# Perfect Single Sign-On Authentication Module [简体中文](README.zh_CN.md)


<p align="center">
    <a href="http://perfect.org/get-involved.html" target="_blank">
        <img src="http://perfect.org/assets/github/perfect_github_2_0_0.jpg" alt="Get Involed with Perfect!" width="854" />
    </a>
</p>

<p align="center">
    <a href="https://github.com/PerfectlySoft/Perfect" target="_blank">
        <img src="http://www.perfect.org/github/Perfect_GH_button_1_Star.jpg" alt="Star Perfect On Github" />
    </a>  
    <a href="http://stackoverflow.com/questions/tagged/perfect" target="_blank">
        <img src="http://www.perfect.org/github/perfect_gh_button_2_SO.jpg" alt="Stack Overflow" />
    </a>  
    <a href="https://twitter.com/perfectlysoft" target="_blank">
        <img src="http://www.perfect.org/github/Perfect_GH_button_3_twit.jpg" alt="Follow Perfect on Twitter" />
    </a>  
    <a href="http://perfect.ly" target="_blank">
        <img src="http://www.perfect.org/github/Perfect_GH_button_4_slack.jpg" alt="Join the Perfect Slack" />
    </a>
</p>

<p align="center">
    <a href="https://developer.apple.com/swift/" target="_blank">
        <img src="https://img.shields.io/badge/Swift-4.0-orange.svg?style=flat" alt="Swift 4.0">
    </a>
    <a href="https://developer.apple.com/swift/" target="_blank">
        <img src="https://img.shields.io/badge/Platforms-OS%20X%20%7C%20Linux%20-lightgray.svg?style=flat" alt="Platforms OS X | Linux">
    </a>
    <a href="http://perfect.org/licensing.html" target="_blank">
        <img src="https://img.shields.io/badge/License-Apache-lightgrey.svg?style=flat" alt="License Apache">
    </a>
    <a href="http://twitter.com/PerfectlySoft" target="_blank">
        <img src="https://img.shields.io/badge/Twitter-@PerfectlySoft-blue.svg?style=flat" alt="PerfectlySoft Twitter">
    </a>
    <a href="http://perfect.ly" target="_blank">
        <img src="http://perfect.ly/badge.svg" alt="Slack Status">
    </a>
</p>



This project is a new candidate feature of Perfect Server since version 3.

Please use the latest version of Swift toolchain v4.0.3.

This package builds with Swift Package Manager and is part of the [Perfect](https://github.com/PerfectlySoft/Perfect) project.

## Acknowledgement

Besides the teamwork of PerfectlySoft Inc., many improvements  and features of this module are suggested / contributed by [@cheer / @Moon1102 (橘生淮南)](https://github.com/Moon1102) and [@neoneye (Simon Strandgaard)](https://github.com/neoneye).

## Objectives

- Independently work without ORMs (although it includes a mini ORM actually) or even databases, while it will supports most Perfect database drivers soon.
- Fast, light-weighted, simple, configurable, secured, scalable and thread-safe.
- Session free: a full application of JWT for the single sign-on authentication to any virtual private clouds.

## Why SSO?

Feature|SSO|LocalAuth|Turnstile
------|----|---------|---------
Security Method|AES|Digest|BlowFish
Configurable Security Method|Yes|N/A|N/A
Password Security Level|Highest|Low|High
Password Shadow Generation|Fast|Fast|Very Slow
Package Layout|One Piece|Scattered|Scattered
Build Configurable|Yes|N/A|N/A
Login Control|JWT|Session|Session
Single Sign-On|Yes|N/A|N/A
Token Renewable|Yes|N/A|N/A
Password Quality Control|Protocol|N/A|N/A
Rate Limiter|Protocol|N/A|N/A
Thread Safe|Yes|N/A|N/A
Database Free/Persistence|Yes|N/A|N/A
Log Readability|JSON Friendly|Plain Text|Plain Text
User Profile Extension|Generic/Strong Typed|Dictionary|Dictionary
Database Customization|Protocol|StORM based|StORM based

## SPM Configuration Note

This library is configurable by environmental variables when building with Swift Package Manager. See description below for the settings.

### Database Driver Specification

use `DATABASE_DRIVER` for database driver specifications. If null, the library will build with all compatible databases.

For example,  `export DATABASE_DRIVER=SQLite` will apply a `PerfectSQLite` driver if `swift build`

Currently configurable database drivers are:

Driver Name| Description|Example Configuration
------------|--------------|-----------
JSONFile|a native JSON file based user database|export DATABASE_DRIVER=JSONFile
SQLite|Perfect SQLite|export DATABASE_DRIVER=SQLite
MySQL|Perfect MySQL|export DATABASE_DRIVER=MySQL
MariaDB|Perfect MariaDB|export DATABASE_DRIVER=MariaDB
PostgreSQL| Perfect PostgreSQL|export DATABASE_DRIVER=PostgreSQL

### Local Mirror

This library can use [Perfect Local Mirror](https://github.com/PerfectlySoft/Perfect-LocalMirror) with `URL_PERFECT` to speed up the building process.

For example, `export URL_PERFECT=/private/var/perfect` will help build if install the Perfect Local Mirror by default.


## Quick Start

### Package.Swift

``` swift
.Package(url: "https://github.com/RockfordWei/Perfect-SSO.git", 
majorVersion: the_latest_release)
```

### Import

The first library to import should be `PerfectSSOAuth`, then import the user database driver as need:

Import| Description
------------|--------------
import UDBJSONFile|User database driven by a native JSON file system
import UDBSQLite|User database driven by SQLite
import UDBMySQL|User database driven by MySQL
import UDBMariaDB|User database driven by MariaDB
import UDBPostgreSQL|User database driven by PostgreSQL

### Runtime Initialization

**NOTE** If there is no other libraries, such as Perfect-HTTP Server, to initialize the Perfect-Crypto lib, you would have to manually turn it on:

``` swift
_ = PerfectCrypto.isInitialized
```

### Customizable Profile

Perfect-SSO is a using generic template class to deal with database and user authentications, which means you **MUST** write your own user profile structure.

To do this, design a `Profile: Codable` structure first, for example:

``` swift 
struct Profile: Codable {
  public var firstName = ""
  public var lastName = ""
  public var age = 0
  public var email = ""
}
```

You can put as many properties as possible to this `Profile` design, with **NOTES** here:
- DO **NOT** use `id`, `salt` and `shadow` as property names, they are reserved for the user authentication system.
- Other reserved key words in SQL / Swift should be avoided, either, such as "from / where / order ..."
- The whole structure should be **FLAT** and **FINAL**, because it would map to a certain database table, so recursive or cascaded definition is invalid.
- The length of `String` type is subject to the database driver. By default, the ANS SQL type mapping for a Swift `String` is `VARCHAR(256)` which is using by UDBMariaDB, UDBMySQL and UDBPostgreSQL, please modify source `DataworkUtility.ANSITypeOf()` if need.

### Open Database

Once got the Profile design, you can start database connections when it is ready. Please **NOTE** it is required to pass a sample `Profile` instance to help the database perform table creation.

Database|Example
--------|---------
JSONFile|`let udb = try UDBJSONFile<Profile>(directory: "/path/to/users")`
SQLite|` let udb = try UDBSQLite<Profile>(path: "/path/to/db", sample: profile)`
MySQL|`let udb = try UDBMySQL<Profile>(host: "127.0.0.1", user: "root",password: "secret", database: "test", sample: profile)`
MariaDB|`let udb = try UDBMariaDB<Profile>(host: "127.0.0.1", user: "root",password: "secret", database: "test", sample: profile)`
PostgreSQL|`let udb = try UDBPostgreSQL<Profile>(connection: "postgresql://user:password@localhost/testdb", sample: profile)`

**NOTE** For databases such as SQLite, MySQL, MariaDB and PostgreSQL, drivers will setup two tables - "users" and "tickets" - for management purposes.

### Log Settings

Login / Access control is always sensitive to log file / audition. 

You can use an existing embedded log file driver `let log = FileLogger("/path/to/log/files")` or write your own log recorder by implement the `LogManager` protocol:

``` swift
public protocol LogManager {
  func report(_ userId: String, level: LogLevel, 
  event: LoginManagementEvent, message: String?)
}

```

**NOTE** The log protocol is assuming the implementation is thread safe and automatically time stamping. Check definition of `FileLogger` for the protocol implementation example.

The default `FileLogger` can generate JSON-friendly log files by the calendar date, e.g, "/var/log/access.2017-12-27.log". There is an example of log content:

``` JSON

{"id":"d7123fcf-64f2-4a6d-9179-10e8b227d39b","timestamp":"2017-12-27 12:04:03",
"level":0,"userId":"rockywei","event":5,"message":"profile updated"},

{"id":"7713e1bc-699e-4fd4-aea7-ccf337f0d4bb","timestamp":"2017-12-27 12:04:03",
"level":0,"userId":"rockywei","event":0,"message":"retrieving user record"},

{"id":"3ed0fcb7-dc70-4148-a86b-e9abc2862950","timestamp":"2017-12-27 12:04:03",
"level":0,"userId":"rockywei","event":4,"message":"user closed"},

{"id":"5f80cf7b-a902-4a66-9665-a885734d9005","timestamp":"2017-12-27 12:04:49",
"level":0,"userId":"rockywei","event":1,"message":"user registered"},

{"id":"56cde3cd-d4bf-4af3-a852-8c6c6a2f3f85","timestamp":"2017-12-27 12:04:49",
"level":0,"userId":"rockywei","event":0,"message":"user logged"},

{"id":"cb49d385-1d64-44b3-9843-f399dcfbd1ee","timestamp":"2017-12-27 12:04:49",
"level":0,"userId":"rockywei","event":0,"message":"retrieving user record"},

{"id":"00f72022-0b8e-422f-9de9-82dc6059e399","timestamp":"2017-12-27 12:04:49",
"level":1,"userId":"rockywei","event":0,"message":"access denied"},

```

The log level and log event are defined as:

``` swift
public enum LoginManagementEvent: Int {
  case login = 0
  case registration = 1
  case verification = 2
  case logoff = 3
  case unregistration = 4
  case updating = 5
  case renewal = 6
  case system = 7
}

public enum LogLevel: Int {
  case event = 0
  case warning = 1
  case critical = 2
  case fault = 3
}
```

### Login Manager

A Login Manager can utilize the user database driver and log filer to control the user login:

``` swift
let man = LoginManager<Profile>(udb: udb, log: log)
```

Certainly, you can skip the log system if no need, which will log to the console indeed:

``` swift
let man = LoginManager<Profile>(udb: udb)
```

Now you can use the `LoginManager` instance to register, login, load profile, update password and update profile, or drop users: 

#### Register & Login

``` swift
// register a user by its id
try man.register(id: "someone", password: "secret", profile: profile)

// generate a JWT token to perform single sign on
let token = try man.login(id: "someone", password: "secret")
```

**NOTE**: By default, both user id and password are in a length of **[5,80]** string. 
Check [Login / Password Quality Control](#login--password-quality-control) for more information.

The token generated by `LoginManager.login()` is a JWT for HTTP web servers.

It is supposed to send to the client (browser) as a proof of authentication.

Besides, `login()` function also provides a carrier argument to allow extended information once logged on:

``` swift
let token = try man.login(id: "someone", password: "secret", header: ["foo": "bar"])
```

#### Token Verification

Every once the client sent it back to your server, `LoginManager.verify()` should be applied to verify the token.

It will automatically validate the content inside of the token, such as  however, it will also return the header and content dictionary as well

It is especially useful when single sign-on is required: set the `allowSSO = true` to forward the token issuer checking from internal to  the end user,
Otherwise, `verify` will reject any foreign issuers, even the token itself is valid.

``` swift
let (header, content) = try man.verify(token: token_from_client, allowSSO: true)

guard let issuer = content["iss"] as? String {
  // something wrong
}

if issuer == man.id {
  print("it is a local token.")
} else {
  print("it is a foreign token.")
}

// header and content are both valid dictionaries, can apply further validation.
```

#### Token Renewal

In certain cases, the token needs to be renewed before further operations.

LoginManager provides a `renew()` function, not only for renewal,
but also allows the developer to add / replace any information by passing new headers, in a form of dictionary:

``` swift
let renewedToken = try man.renew(id: "someone", headers: ["foo":"bar", "fou": "par"])

// the renewed token is different from the previous issued tickets,
// but it can be verified in the same interface.
```

#### Log out

Login manager also provides an optional parameter inside of the `verify` function interface to cancel a previously issued token:

``` swift
let (header, content) = try man.verify(token: token, logout: true)

// if success, both header and content value is still be readable,
// but the token is no longer valid.
```

**NOTE** JWT is not supposed to logout in RFC7519, however, Perfect-SSO is using a "blacklist" to ban those "logoff" tickets, which is practical and could be propagated to all sites by sharing the "blacklist" (in database, the "tickets" table).

#### Load User Profile 

You can retrieve the user profile by its id:

``` swift 
 let profile = try man.load(id: username)
```
#### Update Password

``` swift
try man.update(id: username, password: new_password)
```

**NOTE**: By default, both user id and password are in a length of **[5,80]** string.
Check [Login / Password Quality Control]((#login--password-quality-control)) for more information.

#### Update Profile

``` swift
try man.update(id: username, profile: new_profile)
```

#### Drop A User

``` swift
try man.drop(id: username)
```

## Advanced LoginManager Configuration

The full configuration of `LoginManager` is listed in the class definition:

``` swift
/// a generic Login Manager
public class LoginManager<Profile> where Profile: Codable {

  public init(cipher: Cipher = .aes_128_cbc, keyIterations: Int = 1024,
  digest: Digest = .md5, saltLength: Int = 16, alg: JWT.Alg = .hs256,
  udb: UserDatabase,
  log: LogManager? = nil,
  rate: RateLimiter? = nil,
  pass: LoginQualityControl? = nil,
  recycle: Int = 0)
}
```

### Encryption Control

The first section of `LoginManager` constructor is the encryption control:

- cipher: a cipher algorithm to do the password encryption. AES_128_CBC by default.
- keyIterations: key iteration times for encryption, 1024 by default.
- digest: digest algorithm for encryption, MD5 by default.
- saltLength: length to generate the salt string, 16 by default.
- alg: JWT token generation algorithm, HS256 by default

### User Database Driver

Please check [Open Database](#open-database) for more information

### Log Manager

Please check [Log Settings](#log-settings) for more information


### Rate Limiter

A `RateLimiter` is a protocol that monitors unusual behaviour, such as excessive access, etc.
The login manager will try these callbacks prior to the actual operations
You can develop your own rate limiter by implementing the protocol below:

``` swift
public protocol RateLimiter {
  func onAttemptRegister(_ userId: String, password: String) throws
  func onAttemptLogin(_ userId: String, password: String) throws
  func onLogin<Profile>(_ record: UserRecord<Profile>) throws
  func onAttemptToken(token: String) throws
  func onRenewToken<Profile>(_ record: UserRecord<Profile>) throws
  func onUpdate<Profile>(_ record: UserRecord<Profile>) throws
  func onUpdate(_ userId: String, password: String) throws
  func onDeletion(_ userId: String) throws
}
```

Check the table below for callback event explanation.
**NOTE** Implementation should yield errors if reach the limit.

Callback Event|Description
-----------------|-------------
onAttemptRegister|an attempt on registration
onAttemptLogin|an attempt on login, prior to an actual login.
onLogin|a login event, after a successful login
onAttemptToken|an attempt on token verification, prior to an actual verification
onRenewToken|a token renew event
onUpdate|a user profile update event, could be update password or profile
onDeletion|an attempt on deletion user record

### Login / Password Quality Control

`LoginManager` also accepts customizable login / password quality control.
If the protocol is implemented, it will call these interfaces as events when necessary.
The verification is not limited to typical scenarios such as registration or password update to avoid weak credentials, but also applicable in routine user name checking, especially as protecting the login system from oversize username cracks.

``` swift
public protocol LoginQualityControl {
  func goodEnough(userId: String) throws
  func goodEnough(password: String) throws
}
```

### Token Recycler

`LoginManager` can cancel any previously rejected tickets. You can customize the token recycling time span by set the `recycle` parameter, in seconds, and 60 by default.

**NOTE** a smaller recycling timeout may result in a bigger system usage.

## Customize Your Own Database Drivers

If you want to implement a different database driver, just make it sure to comply the `UserDatabase` protocol:

``` swift
/// A general protocol for a user database, UDB in short.
public protocol UserDatabase {

  /// -------------------- BASIC CRUD OPERATION --------------------
  /// insert a new user record to the database
  func insert<Profile>(_ record: UserRecord<Profile>) throws

  /// retrieve a user record by its id
  func select<Profile>(_ id: String) throws -> UserRecord<Profile>

  /// update an existing user record to the database
  func update<Profile>(_ record: UserRecord<Profile>) throws

  /// delete an existing user record by its id
  func delete(_ id: String) throws
  
  /// -------------------- JWT TOKEN MANAGEMENT SYSTEM --------------------
  /// put a ticket with its expiration to the blacklist
  func ban(_ ticket: String, _ expiration: time_t) throws

  /// test if the giving ticket has been banned.
  func isRejected(_ ticket: String) -> Bool
}

```

Please feel free to check the existing implementation of UDBxxx for examples.

## Notes

More examples can be found in the test script.

## Issues

We are transitioning to using JIRA for all bugs and support related issues, therefore the GitHub issues has been disabled.

If you find a mistake, bug, or any other helpful suggestion you'd like to make on the docs please head over to [http://jira.perfect.org:8080/servicedesk/customer/portal/1](http://jira.perfect.org:8080/servicedesk/customer/portal/1) and raise it.

A comprehensive list of open issues can be found at [http://jira.perfect.org:8080/projects/ISS/issues](http://jira.perfect.org:8080/projects/ISS/issues)

## Further Information
For more information on the Perfect project, please visit [perfect.org](http://perfect.org).
