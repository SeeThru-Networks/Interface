# Introduction
The Feed Interface is the command line method of uploading a set of feed results to SeeThru Networks for distribution to users.  
We handle the process of feed result distribution via our platform which allows you to focus on creating feeds.  

## Feeds
A feed is a test that get's executed and once executed sends the result of the test to the SeeThru platform via a feed interface.  

The fundamental principle of a feed is that it evaluates to a `color`, a `message` and a `time`.
- The `color` is a color result of the test, `red`, `amber`, `green`.  
- The `message` is a message that goes along with the status to give a little more information about the feed result.  
- The `time` is the time at which the test got executed.  

The purpose of this evaluation is to provide a clear representation of a test result to an end user.  

An end user can pick to see a feed on their device via the SeeThru platform, and once a feed is chosen, the cloud will color to the `color` given and the `message` shown (if any) of the latest feed result uploaded to the SeeThru platform.  

### Information
- A feed has a feed guid, which is used to identify the feed.  
- A feed belongs to a Feed Interface, this is configured via the SeeThru platform.  

## Feed Interface - Concept
**Note:** There is a distinction between a Feed Interface and the Feed Interface command line tool, here we are talking about the concept of a feed interface.  

A Feed Interface is the method of sending a set of feed results to the SeeThru platform. **All** feeds belong to a feed interface and **all** feed results get uploaded to the SeeThru platform via the Feed Interface.    

The Feed Interface not only allows for grouping of similar feeds under a single feed interface but it also handles the secure transmission of feed results to the SeeThru platform. It does this via:
- An `access_token` which is the identifier of the feed interface.  
- A `secret` which provides authentication of the feed interface (this **must** be kept secret).  

Upon connection of a Feed Interface to the SeeThru platform. The feed interface is authenticated using the `access_token` and `secret`. After a feed interface is authenticated, any results of feeds that belong to the Feed Interface can be uploaded.

### Information
- Many feed results can be uploaded for a single feed however only the latest feed result is stored.  
- Feed Results uploaded will **only** be stored once the Feed Interface ends the connection to the SeeThru platform.  
- All feed results are sent encrypted to the SeeThru platform.  
- After connecting to the SeeThru platform, a Feed Interface can't connect again for a set amount of time (by default this is 45 seconds).  

## Feed Interface - Command Line Tool
The Feed Interface command line tool is the tool that this document is for.  

The tool handles the Feed Interface process and uploading of feed results using a configuration file.  

When the tool is used, you need to give it a configuration file. This configuration file identifies the `access_token`, `secret` and any feeds of the feed interface. Any feeds in the configuration file **must** belong to the feed interface.  

### Information
- Every time you want to upload feed results to the SeeThru platform, this tool can be used.  
- A feed interface is identified by the configuration file, therefore you can use many feed interfaces with the same tool.  

# The config file
- The config file tells the tool where to find all of your feeds and any authentication arguments, every config file is unique to a separate feed interface created on the SeeThru platform.  
- The file extension of the config file is `.conf`  
- Every property in the config file **must** be surrounded by `"` in order for the property to be valid.  
- The format for a property in the config file is as follows: `property_name="property_value"`.  
- Every property **must** follow ascii text encoding in order for it to be valid.  
  
### Required properties
- `secret` - This is the secret key given to you by the SeeThru platform when you create your feed interface, this is a series of 64 hex characters.  
- `access_token` - This is the access token given to you by the SeeThru platform when you create your feed interface, this is a series of 32 hex characters represented as 36 characters using dashes.  

### Optional properties
- `base_dir` - If this is given, the value should be a directory that all paths defined in the configuration file should extend from.  

### Feeds
Every feed in the config file is represented by a block, each block must begin with the `[External]` header. The block has the following arguments:
- `guid` - **Required** - This is the feed guid given to you when you create a feed on the SeeThru platform, this feed **must** belong to the feed interface defined with the access_token.  
- `name` - Optional - This names the feed in the config file for easy identification of a feed, this isn't used by the tool.  
- `source` - **Required** - This tells the tool where to find the result for the feed, the format of this result is defined in the result section.  
- `version` - **Required** - This defines the version of the feed, this will be sent to the SeeThru platform, the version identifier can be a **maximum** of 8 characters.  
  
### Example
An example config is as follows:  

```
secret="Your_Secret_Key_From_The_SeeThru_Platform"  
access_token="Your_Access_Token_From_The_SeeThru_Platform"  
  
[External]  
guid="Your_Guid_0"  
name="Feed_0"  
source="/Path/To/Feed/Result_0.json"  
version="1a"  

[External]  
guid="Your_Guid_1"  
name="Feed_1"  
source="/Path/To/Feed/Result_1.json"  
version="5c"  
```  

# The feed result
The feed result for the tool should be given as a json file, the source in the configuration file for a feed block should point to this json file.  

The json file should contain three properties, `color`, `message` and `time`.  
- The `color` property defines the overall result of a test, `green` means good, `red` means bad and `amber` is somewhere in between, you choose the conditions to make a color in your test.  
- The `message` property is an accompanying message with the color. An example of this would be that with a `red` color, the message would describe the exact error that occurred in your test.  
- The `time` property defines the time at which the test was performed.  

### Formatting
All properties have string types.  
The color property can take the following values:  

- `green`  
- `amber`  
- `red`  

The `color` property must be formatted as shown above, i.e. all lowercase and without extra extraneous characters.  
The `message` property **must** be **less** than 256 characters for it to be valid and can be any message you would like.  
The `time` property **must** be **less** than 64 characters and **must** be in the format `YYYY-MM-DD hh:mm:ss`.  

The properties should all be stored in the top level of the json:
```
{
  "color": "red",
  "message": "The port on the server couldn't be reached",
  "time": "1970-01-01 00:00:00"
}
```  

# Running
A recommended implementation of the Feed Interface where the test runs periodically would be as cron.  
This is so that all test results can be uploaded soon after they completed.  

- If the test result needs to be uploaded immediately after a test has completed, please look at using the Feed Interface library,
this is a c library that directly communicates with the SeeThru server and can be used in your app.  

A recommended cron would be:
`*/2 * * * * /Path/To/Feed/Interface/FeedInterface -c /Path/To/Config/File/config.conf -lO /var/log/FeedInterface.log -d 5`  
This will run the tool 5 seconds into every minute, directing the logs to the path shown.  
The arguments possible are:  

- `-c or --config` - **Required** - This defines the path to the config file  
- `-lO or --log` - Optional - This defines the file to store the logging output of the Feed Interface, **Note:** This redirects the output stream to that file and so there is no stdout output.  
- `-d or --delay` - Optional - This defines the time after the executable is ran to actually start execution, in seconds. This is used with cron to start execution part way into the cron.
e.g. A delay of 5 would start 5 seconds into the cron to allow for all tests that may be on the same cron to execute first.  
  
A standalone execution of the Feed Interface would be:
`/Path/To/Feed/Interface/FeedInterface -c /Path/To/Config/File/config.conf`  
  
Execution works the same on both Windows, Linux and macOS. Note: Please use the correct binaries on each platform.  

# Error Codes
There are a set of possible error codes that can be shown, these are as follows:  

| Error Code    | Meaning                                       |
| ------------- |:---------------------------------------------:|
| -1            | Server Disconnect                             |
| 1             | DNS Lookup Failed                             |
| 2             | Cannot Connect to server                      |
| 5             | No entry                                      |
| 6             | No Feeds for Interface                        |
| ------------- | -------------									|
| **From Server**|                                              |
| 0x01          | ServerDisconnect                              |
| 0             | NoError                                       |
| 0x80          | ForceDisconnect                               |
| 0x81          | ServerDownForMaintenance                      |
| 0x82          | ConnectionAttemptBeforeTimeout                |
| 0x90          | AccessTokenNotLongEnough                      |
| 0x91          | InvalidSecret                                 |
| 0x92          | NoInterfaceEntry                              |
| 0x93          | NoInterfaceFeeds                              |
| 0xA0          | InvalidCharInMessage                          |
| 0xA1          | InvalidCharInTimestamp                        |
| 0xA2          | InvalidCharInVersion                          |
| 0xA3          | MessageTooLong                                |
| 0xA4          | TimestampTooLong                              |
