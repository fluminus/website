---
layout: post
title: LumiNUS API for Dart development guide
date: 2019-05-24 20:20 +0800
categories: fluminus dart
author: Yuanhong
---

Since Fluminus is an app heavily dependent on LumiNUS APIs, from the very beginning I felt necessary extracting the API-relevant part into a separate package for better upgradability and maintainability. After implementing the core functionality (i.e. authentication and raw API calls), I believe it is time that I write this tutorial of expanding the capability of [luminus_api](https://pub.dev/packages/luminus_api) package by referring to [the official API specification](https://luminus.portal.azure-api.net/docs/services) since most of the work is procedural.

<!--more-->

<style>
.gist-data{
    height:250px; // Any height
    overflow-y: visible;
}
</style>

## A Quick Look At Source Code

Let's look at one specific API `getProfile`. There are two parts to this API - one in `luminus_api.dart` which is exposed to the package users, another under `src` folder named `profile.dart` which stores the data abstraction of the response of this API call.

#### Inside `luminus_api.dart` we have:

```dart
export 'package:luminus_api/src/profile.dart';
import 'package:luminus_api/src/profile.dart';

// [other imports and exports omitted]

class API {

// [other properties and methods ommitted]

    /// Returns a [Profile] object
    static Future<Profile> getProfile(Authentication auth) async {
        Map resp = await API._rawAPICall(auth: auth, path: "/user/profile");
        var profile = new Profile.fromJson(resp);
        return profile;
    }

}
```

The structure is basically the same, first get `resp` by calling the raw API path; then dump the received JSON into the constructor of our data abstraction class, return the object representing the response data in the end.

#### Inside `profile.dart` we have: 

(I omitted most of these code because we will generate them using an online tool later)

```dart
class Profile {
  String id;
  // omitted other fields

  Profile(...);

  Profile.fromJson(Map<String, dynamic> json) {
      ...
  }

  Map<String, dynamic> toJson() {
      ...
  }
}
```

Methods of generating the data abstraction class of JSON response will be shown later.

## A Step-by-step Guide Of Translating One API

Let's look at [Get All Announcements](https://luminus.portal.azure-api.net/docs/services/announcement/operations/GetActiveAnnouncements?) API. Note that for now we will only focus on APIs under **GET** method since information retrieval is our main focus as of now.

### Base URL

We see the request URL is [https://luminus.azure-api.net/announcement/Active[?sortby][&offset][&limit][&where][&populate][&titleOnly]](). This is easy to understand, the final request URL following this formula should look like: 

- [https://luminus.azure-api.net/announcement/Active]()
- or [https://luminus.azure-api.net/announcement/Active?sortby=time]()
- or [https://luminus.azure-api.net/announcement/Active?limit=10]()
- or [https://luminus.azure-api.net/announcement/Active?limit=10&sortby=time&titleOnly=True]()

You should get the idea by now. What's nicer is that you don't need to provide [https://luminus.azure-api.net]() part to my function `rawAPICall`, you only need to provide the rest as the `path` parameter. 

Apparently, the "root" of this API is [/announcement/Active](), while the rest are all parameters.

### Query parameters

![Query params](https://i.imgur.com/Yz1JfjU.png)

As you can see here, all parameters are optional, so we could write our function signature like this:

```dart
/// Get active announcements
/// Populate additional information. Accepted entities: creator, lastUpdatedUser, parent
static Future<List<Announcement>> getActiveAnnouncements(Authentication auth,
    {String sortby,
    int offset,
    int limit,
    String where,
    String populate,
    bool titleOnly = true}) async {
  // [...later...]
}
```

Note that as the parameters are optional, we're using the optional parameter in Dart. In Dart, optional parameters without assignment will be `null` by default, those with an assignment are called *optional parameters with default values*. Note that we're sticking to the API specification, meaning that `titleOnly` is *True* by default.

Then we should generate the path including these parameters, just follow this template:

```dart
String query = formatQueryArgument('sortby', sortby) +
    formatQueryArgument('offset', offset) +
    formatQueryArgument('limit', limit) +
    formatQueryArgument('where', where) +
    formatQueryArgument('populate', populate);
String path = '/announcement/Active?titleOnly=${titleOnly}' + query;
Map resp = await API._rawAPICall(auth: auth, path: path);
// [...later...]
```

Note that I provided the function `formatQueryArgument` to help you leave our parameters that are `null`, and since `titleOnly` will never be `null` unless you're bored enough to pass a null as the parameter (well this might a bug I could fix later), it's hard-coded in the path.

What if all parameters are optional? You just leave the question mark there as I believe LumiNUS API server is smart enough to ignore it (not experimented, don't believe my words). Like this `String path = '/announcement/Active?' + query;`

### Data abstraction class

**Note:** Don't actually overwrite my `announcement.dart` file because you're going to find what you are going to produce will likely be the same as what I have done. So save yourself a few minutes on this. But it's always good to check that the response is consistent, maybe LumiNUS will response different announcement objects? I don't know... Just be careful. Also, the API specification for the response is not accurate. Don't believe that.

Add `import 'package:dotenv/dotenv.dart' show load, env;` to the beginning of `luminus_api.dart`, create a `main` function inside the file like this:

```dart
main(List<String> args) async {
  load();
  print(jsonEncode(await API._rawAPICall(
      auth: Authentication(
          password: env['LUMINUS_PASSWORD'], username: env['LUMINUS_USERNAME']),
      path: '/announcement/Active?limit=1')));
}
```

In your console, you should read something like this:

```json
{"status":"success","code":200,"total":104,"offset":0,"limit":1,"data":[{"id":"8f164b0c-858b-4c12-b79a-0680c09b6b61","parentID":"063773a9-43ac-4dc0-bdc6-4be2f5b50300","title":"Exam answers and workings","description":"balabala","displayFrom":"2019-05-03T19:49:21.65+08:00","expireAfter":"2019-05-31T19:49:00+08:00","archiveAfter":"2019-08-31T19:49:21.65+08:00","publish":true,"sms":false,"email":false,"access":{"access_Full":false,"access_Read":true,"access_Create":false,"access_Update":false,"access_Delete":false,"access_Settings_Read":false,"access_Settings_Update":false},"createdDate":"2019-05-03T20:09:53.21+08:00","creatorID":"3e3fa871-f4ef-4051-9208-e1207354804c","lastUpdatedDate":"2019-05-03T20:09:53.21+08:00","lastUpdatedBy":"3e3fa871-f4ef-4051-9208-e1207354804c"}]}
```

Copy and paste this onto [JSON to Dart Tool](https://javiercbk.github.io/json_to_dart/), remove the square brackets at the beginning and end, click **Generate Dart**, and you get this:

<script src="https://gist.github.com/le0tan/8f8219a6e54b7ecea85f0ffe92199358.js"></script>

Change all *Autogenerated* to *AnnouncementResponse*. 
Change *Data* to *Announcement* and cut the *Announcement* class to a separate file named `announcement.dart` under `src` folder.
In `announcement.dart`, remove *Access* class and import `/src/access.dart` instead since this class appeared repeatedly in responses.

### Finishing up...

Last step! Let's go back to our API function and finish it!

```dart
static Future<List<Announcement>> getActiveAnnouncements(Authentication auth,
    {String sortby,
    int offset,
    int limit,
    String where,
    String populate,
    bool titleOnly = true}) async {
  String query = formatQueryArgument('sortby', sortby) +
      formatQueryArgument('offset', offset) +
      formatQueryArgument('limit', limit) +
      formatQueryArgument('where', where) +
      formatQueryArgument('populate', populate);
  String path = '/announcement/Active?titleOnly=${titleOnly}' + query;
  Map resp = await API._rawAPICall(auth: auth, path: path);
  var announcements = new AnnouncementResponse.fromJson(resp);
  return announcements.data;
}
```

And maybe you could do me a favor and test the function a little bit by change the `main` to:

```dart
main(List<String> args) async {
  load();
  var auth = Authentication(
          password: env['LUMINUS_PASSWORD'], username: env['LUMINUS_USERNAME']);
  print(await API.getActiveAnnouncements(auth, limit: 1));
}
```

The output should be something like:

```
[[Announcement] title: Exam answers and workings]
```

or simply `[Object of Announcement]`, depending on whether you're using the version where I overridden `toString` method of `Announcement` or not.

That's all! Now you're ready to contribute to [luminus_api](https://pub.dev/packages/luminus_api)! Please send your pull request [here](https://github.com/fluminus/luminus_api). Thanks a lot!