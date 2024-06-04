# BeeScan 1.0.0
# [DevForum](https://devforum.roblox.com/t/beescan-100-instance-scanner-for-backdoor-detection/3004906) | [Roblox](https://create.roblox.com/store/asset/5240105231/BeeScan)

# About
BeeScan is powerful game instance scanner for backdoors detection. It is a one click solution that most developers can use to get a quick health check on their games. It's designed to detect hidden scripts, obfuscation attempts, and illegal access to global functions by reading the script content. **To make this system passive, this plugin does NOT modify/remove scripts.** It is the users job to resolve the flags (usually by deleting the script).

# Features:

### Known Solutions
There are already solutions out there, such as [SteadyOn’s Instance Scanner](https://create.roblox.com/store/asset/2256681875/SteadyOns-Instance-Scanner), [Server Defender](https://create.roblox.com/store/asset/381046418/Server-Defender-OFFICIAL-PLUGIN), or [Kronos](https://devforum.roblox.com/t/kronos-scan-your-game-for-viruses-backdoors-plugin/224779). But they either are out of date or lack certain features given below.

### Multiple Layers of Scanners
BeeScan has multiple scanners  that run along a pipeline. All it takes it for 1 scanner to flag a problem. Because there are multiple scanners in a pipeline, it's easy for you to add your own processing systems.

```text
  -------------------------- BEGAN SCAN [1.0.0] --------------------------
  [MaterialService.bad] caught by [OutlierCheck]:
		--> Script RunContext is set to Enum.RunContext.Server instead of Legacy so script is allowed to run anywhere, please review

		--> bad is a ServerScript but is residing in a blacklisted parent MaterialService

		


  [Workspace.Malware.obfuscate] caught by [FlexRules]:
		Obfuscation attempt found, method is reassigning a lua GLOBAL (loadstring)
		--> local x = loadstring

		Script uses loadstring, please verify it's authorized and mark as good
		--> local x = loadstring
		


  scanned 2 instances
  -------------------------- SCAN COMPLETE --------------------------
```


### Smart String Pattern Source Code Matching
One of the unique scanners that run with BeeScan is FlexRules, a rule based script pattern detector. Manual rules are generated and entered into the rule library, which will try to match the pattern with real world scripts. The system is smart enough to ignore comments and whitelisted samples.

```lua
local x = require -- flagged
local x = require(123456789) -- flagged
local x = require(workspace.MyModuleScript) -- allowed
--local x = require(123456789) -- allowed
```

Some common patterns include *require*, *loadstring*, *getfenv*, *setfenv*, *_G*, and *HttpService*.

### Instance Outlier Scanning
The system will also alert you of suspicious scripts. These include those that hide inside hidden services, have really large sizes, or use RunContext. **"game:GetDescendants()" is used for scanning to ensure that new services are also scanned.**

```text
  [VRService.testPrint] caught by [OutlierCheck]:
		--> Script RunContext is set to Enum.RunContext.Server instead of Legacy so script is allowed to run anywhere, please review

		--> "testPrint" is a ServerScript but is residing in a blacklisted parent VRService
```


### Code Verification and Signatures
Verify scripts by "signing" them. Signing will attach a timestamp, your user ID, and a hash the scanner will use to compare with. Changing any part of the script (hash, timestamp, user ID, script name, script content) will invalidate the signature and flag it again.

### Open Source with Modifications Allowed
This project is open sourced. You are free to make your own modules and add it to the pipeline. If you feel generous, please contribute any changes you make! Over time, if there are new methods to backdoor, I will try and update the plugin.

# How to Use

## Quick Use
1. Download plugin through Roblox plugins.
2. Open a roblox studio place you want to check.
3. Click "Scan"

## Development/In-house/Close Loop copy
1. Download plugin through GitHub.
2. Open a roblox studio place you want to check.
3. Load the .rbxm file.
4. To override the hash salt/private key, update BeeScan.Salt.Value, otherwise, leave it blank as "".
5. Right click the BeeScan folder, then click "Save as Local Plugin..."
6. Click "Scan"

### Scan
Scans the selection and prints out the report in output logs.
- deselect everything in workspace and click "Scan" to scan the entire game
- select an Instance and click "Scan" to scan the instance and its descendants
- select multiple Instances and click "Scan" to scan the instances and their descendants

### Hash
Used on scripts only. "Signs" the script using your Roblox UserID and the datetime, then generate a hash. As long as the hash is valid, the scanner will ignore it.
1. find a script that's marked by the scanner as suspicious
2. click on that script, then click "Hash"
3. you've now signed the script and said it's "Safe", your user ID and date/time signed is available for other developers to look at.

### New Salt
**Only available if plugin salt isn't modified.** This will generate a new "private key" for you that you sign with. The scanner, now using this new key, will invalidate all your previously signed scripts. Only click this if you believe your plugin private key has been found, OR you want to sync your private key with a friend.

No selection vs. Selection of an instance before clicking "New Salt"

# Technical Detail (Scanners)
The hash check scanner simply verifies if the hash signature is valid.
Hash generation and user signature is a blend of attributes:
- Script name
- Script source/content
- User ID who verified this
- Date/time
- Your plugin's private key (auto generated)

If any of these are changed, the signature will be invalid and the scanner will pick it up again.

This scanner tries to match for certain code patterns rather than direct entries. The scanner is smart enough to ignore comments. Some checks include keyword checks, variable assignment checking, and common obfuscation string checks. The majority of detections will be done using FlexRules.

This scanner looks at the script properties and checks for any anomalies, such as suspicious Parents, RunContext, and script size. This system will prevent most backdoors from hiding inside hidden services.

To remove scripts from hidden services, you can run a command line, or enable the service to be visible in studio and delete the script.

This scanner simply matches names to a list of virus names. It also detects for deviations in naming format (for example, using unknown characters)
```text
  -------------------------- BEGAN SCAN [1.0.0] --------------------------
  [Workspace.Malware.h�mer�] caught by [NameCheck]:
	h�mer� is using invalid characters!

  scanned 4 instances
  -------------------------- SCAN COMPLETE --------------------------
  Keyword Scanner: Some areas are in need of review. [1 notice]
```

## Threat Model
This plugin is intended for Roblox Studio. This plugin is made to stop bad actors (developers & plugins) from injecting hidden code into your game. It's designed to detect and flag hidden scripts, obfuscation attempts, and illegal access to global functions by reading the script content. This plugin does NOT detect common "troll"-like or self-replicating scripts, such as Anti-Lag and fire spreader due to their signatures overlapping with actual valid scripts.

**This plugin does NOT modify/remove the scripts.** This is because I don't want to disrupt development because of false positives. This plugin simply finds script violations for you. It is your job to remove it.

## Limitations
This plugin, like most security systems, will eventually become out of date unless it's periodically updated.

Do not assume flagged scripts are BAD. Treat flagged scripts as "suspicious" unless you can confirm it's safe. The system tends to be trigger happy.

# FAQ
### My friend hashed a script and verified it was good, but my BeeScan is showing a hash mismatch on that script?
This is normal, the plugin is intentionally designed to not trust anyone. If you trust your friend, both of you will come up with a private key. Create an Instance, like a BasePart, and name it that private key, select it, then click "New Salt". Your output log should show that the salt/private key has been set to a new value.

### Can I use this plugin to scan other plugins?
This plugin can only be used to scan in-studio Instances (things that sit under Explorer). You cannot scan your active plugins directly. You will need to obtain the source file/source code of the plugin, then insert it into workspace/game and use BeeScan on it.

### Can I use this plugin to scan 3rd party systems? For example: how do I know require(123456789) returns a safe plugin?
You would have to find the source code of that plugin, put it in workspace/game and use BeeScan on it.

# Credits
- This plugin uses HashLib by Egor Skriptunoff, boatbomber, and howmanysmall (https://devforum.roblox.com/t/open-source-hashlib/416732/1)
- Some additional devforum posts were used to test the plugin (https://devforum.roblox.com/t/how-does-backdoors-work-nexus-admin-exploit-works-in-detail/2922935, https://devforum.roblox.com/t/how-to-check-for-backdoors/2970564, https://devforum.roblox.com/t/how-to-remove-rosync-virus/2983195)
- Special thanks to Meta_data for testing
