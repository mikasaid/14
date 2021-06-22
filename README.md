# 14
Hello
Provides initial support for installing basic Git Bash fragments for Windows Terminal (see git-for-windows/git/issues/3183 for details).
Known Issues:

Final location of git-bash-global.json and git-bash-local.json in installer repository TBD
Mix and match of globally installed Terminal and locally installed Git (Bash) and vice versa does not work
Custom Git installation directories are not supported in the fragments
Should probably include a pre-generated UUIDv5 according to the Terminal docs for later modification
I am raising this WIP pull request to initiate the discussion on how to fix the aforementioned issues.

@rimrul
rimrul reviewed on May 3
Member
rimrul left a comment
Final location of git-bash-global.json and git-bash-local.json in installer repository TBD

I think they should go into git-extra, but I'm not completely sure.

Mix and match of globally installed Terminal and locally installed Git (Bash) and vice versa does not work

I'm don't think we actually need to care how Windows terminal was installed.

Custom Git installation directories are not supported in the fragments

We might be able to disable the component for those unsupported cases with a check wether the value of {app} is what we expect.

Should probably include a pre-generated UUIDv5 according to the Terminal docs for later modification

Unless I'm missreading the documentation, that's only for modifying profiles that Windows Terminal comes with.

The only profiles that can be modified through fragments are the default profiles, Command Prompt and PowerShell, as well as dynamic profiles.

and "dynamic profiles" refers to these:

Windows Terminal will automatically create Windows Subsystem for Linux (WSL) and PowerShell profiles for you if you have these shells installed on your machine.

So I think the way we're supposed to update our profile is by updating the json file.

installer/install.iss
Outdated
installer/git-bash-local.json
Outdated
installer/install.iss
Outdated
installer/install.iss
Outdated
installer/install.iss
Outdated
@Okeanos
Contributor
Author
Okeanos commented on May 3
General side question: I presume the merge request should be a single commit in the end, i.e. all fixes for the issues squashed into a single commit with a proper description and signed-off-by line?

@rimrul
Member
rimrul commented on May 3 • 
We might also be able to avoid the install directory issues if we dynamically generate the json file like we do with the xml file for the auto updater in InstallAutoUpdater. So something like

procedure InstallWindowsTerminalFragment;
var
    Res:Longint;
    AppPath,XMLPath:String;
begin
    if IsAdminInstallMode() then
        JSONPath:=ExpandConstant('{commonappdata}\Microsoft\Windows Terminal\Fragments\Git\git-bash.json')
    else
        JSONPath:=ExpandConstant('{localappdata}\Microsoft\Windows Terminal\Fragments\Git\git-bash.json');
    AppPath:=ExpandConstant('{app}');
    SaveStringToFile(JSONPath,
        '{'+
        '    "profiles": ['+
        '      {'+
        '        "name": "Git Bash",'+
        '        "commandline": "'+AppPath+'\usr\bin\bash.exe -i -l",'+
        '        "icon": "'+AppPath+'\{#MINGW_BITNESS}\share\git\git-for-windows.ico",'+
        '        "startingDirectory": "%USERPROFILE%"'+
        '      }'+
        '    ]'+
        '  }',False);
end;
might work.

@rimrul
Member
rimrul commented on May 3
I presume the merge request should be a single commit in the end, i.e. all fixes for the issues squashed into a single commit with a proper description and signed-off-by line?

Yes, exactly. just force push your updated commit and this PR will be updated accordingly.

@Okeanos
Contributor
Author
Okeanos commented on May 3
We might also be able to avoid the install directory issues if we dynamically generate the json file like we do with the xml file for the auto updater in InstallAutoUpdater. So something like

procedure InstallWindowsTerminalFragment;
var
    Res:Longint;
    AppPath,XMLPath:String;
begin
    if IsAdminInstallMode() then
        JSONPath:=ExpandConstant('{commonappdata}\Microsoft\Windows Terminal\Fragments\Git\git-bash.json')
    else
        JSONPath:=ExpandConstant('{localappdata}\Microsoft\Windows Terminal\Fragments\Git\git-bash.json');
    AppPath:=ExpandConstant('{app}');
    SaveStringToFile(JSONPath,
        '{'+
        '    "profiles": ['+
        '      {'+
        '        "name": "Git Bash",'+
        '        "commandline": "'+AppPath+'\usr\bin\bash.exe -i -l",'+
        '        "icon": "'+AppPath+'\{#MINGW_BITNESS}\share\git\git-for-windows.ico",'+
        '        "startingDirectory": "%USERPROFILE%"'+
        '      }'+
        '    ]'+
        '  }',False);
end;
might work.

The good news: it works. The bad news: AppPath expands using \ as separator which breaks JSON compatibility and renders the fragment unusable by Terminal.

@rimrul
Member
rimrul commented on May 3
Right. This should work around that:

procedure InstallWindowsTerminalFragment;
var
    Res:Longint;
    AppPath,XMLPath:String;
begin
    if IsAdminInstallMode() then
        JSONPath:=ExpandConstant('{commonappdata}\Microsoft\Windows Terminal\Fragments\Git\git-bash.json')
    else
        JSONPath:=ExpandConstant('{localappdata}\Microsoft\Windows Terminal\Fragments\Git\git-bash.json');
    AppPath:=StringChangeEx(ExpandConstant('{app}'), '\', '/', True);
    SaveStringToFile(JSONPath,
        '{'+
        '    "profiles": ['+
        '      {'+
        '        "name": "Git Bash",'+
        '        "commandline": "'+AppPath+'/usr/bin/bash.exe -i -l",'+
        '        "icon": "'+AppPath+'/{#MINGW_BITNESS}/share/git/git-for-windows.ico",'+
        '        "startingDirectory": "%USERPROFILE%"'+
        '      }'+
        '    ]'+
        '  }',False);
end;
@Okeanos Okeanos force-pushed the Okeanos:main branch from 06ab1b2 to fad7008 on May 3
@Okeanos
Contributor
Author
Okeanos commented on May 3
So, I just pushed the changes you suggested:

 Single option install
 Additional Checks
 Custom Procedure with embedded JSON
I tested this with local admin user by installing Git with the local installer (from scratch). Verified installation by opening Terminal and starting the newly registered Git Bash. To finish I also verified the fragment was deleted from disk when Git was uninstalled.

I could set up a local user (non-admin) in my VM as well if desired to test that part of the installation but not today anymore.

@rimrul
Member
rimrul commented on May 3
I could set up a local user (non-admin) in my VM as well if desired to test that part of the installation but not today anymore.

That would be great. Thank you.

@Okeanos
Provide Windows Terminal Fragment Extension …
3158fdb
@Okeanos Okeanos force-pushed the Okeanos:main branch from fad7008 to 3158fdb on May 4
@Okeanos
Contributor
Author
Okeanos commented on May 4
I updated the procedure a little to log errors in case the file could not be saved. That aside the tests (clean installs for admin and non-admin) users as described before worked perfectly fine and the Fragment was successfully detected by Windows Terminal.

@Okeanos Okeanos requested a review from rimrul on May 10
@dscho
dscho approved these changes on May 18
Member
dscho left a comment
Most excellent!

@dscho dscho dismissed rimrul’s stale review on May 18
@Okeanos addressed the concerns

@dscho dscho changed the title WIP: Provide Windows Terminal Fragment Extension installer: provide Windows Terminal Fragment Extension on May 18
@dscho dscho merged commit 94620cd into git-for-windows:main on May 18
1 check passed
dscho added a commit that referenced this pull request on May 18
@dscho
Mention New Feature in release notes …
5e4278f
@zadjii-msft zadjii-msft mentioned this pull request on May 18
Upon installation, add a Git Bash profile, if Git for Windows is present microsoft/terminal#1394
 Closed
@TBBle
TBBle reviewed on May 19
installer/install.iss
@TBBle TBBle mentioned this pull request on May 19
Use /bin/bash when launching from Windows Terminal #348
 Merged
@jeremyd2019
Contributor
jeremyd2019 commented 15 days ago
Should probably include a pre-generated UUIDv5 according to the Terminal docs for later modification

Unless I'm missreading the documentation, that's only for modifying profiles that Windows Terminal comes with.

I think providing a pre-generated UUIDv5 would allow the user to modify/override/disable the profile shipped here in their own settings json. That was my reading of the docs anyway

@DHowett
DHowett commented 14 days ago
You're correct! Providing a stable guid of any sort will allow the user to customize git-for-windows once and have their customizations stick around even if you change the name of the profile. It will also make sure that you collide with yourself (which may or may not be good, depending on your view.)

@jeremyd2019
Contributor
jeremyd2019 commented 14 days ago
So in this case should it be in the "guid" property like in the normal settings, or is the uuid supposed to be called "updates" in the fragment extensions schema?

@315mohsen
This comment was marked as off-topic.
Sign in to view
@dscho
Member
dscho commented 13 days ago
So how would such a GUID be provided? Could you open a PR to do that?

@jeremyd2019
Contributor
jeremyd2019 commented 13 days ago
From microsoft/terminal#10374 (comment)

UGH. Our documentation here is not correct¹.

The namespace GUID is actually based on the name of the folder/fragment name you're using, and it will be used to automatically generate a GUID if you haven't specified one.

Okay -- we can't change this because now we'll break compatibility with all user customizations on top of Fragments (grrr).

Here is the code that will generate the same GUID as Git Bash is already getting. You'll want to use this guid specifically, because doing anything else will separate the user's settings from your upstream document.

import uuid

# CALCULATE NAMESPACE BASED ON FRAGMENT FOLDER NAME
u = uuid.uuid5(uuid.UUID("{f65ddb7e-706b-4499-8a50-40313caf510a}"), "Git".encode("UTF-16LE").decode("ASCII"))

# CALCULATE FINAL BASED ON NAMESPACE
uuid.uuid5(u, "Git Bash".encode("UTF-16LE").decode("ASCII"))

# For Git/Git Bash: UUID('2ece5bfe-50ed-5f3a-ab87-5cd4baafed2b')
¹ The namespace GUID it provides is actually for the internal "Windows.Terminal.Wsl" namespace... which due to legacy reasons doesn't follow the same rules. It was authored in v0.5, and we didn't want to break user customizations back then either. 😬

So we at least now know what the GUID should be. I at least am not sure what "property" it should be on, "guid" or "updates", for a fragment.

@DHowett
DHowett commented 13 days ago
So we at least now know what the GUID should be. I at least am not sure what "property" it should be on, "guid" or "updates", for a fragment.

I'll also make sure the docs clear this up 😄

When the fragment is creating a totally new profile, it should have a guid or a name (the guid will be automatically generated based on the namespace rules above if guid is not provided.). Providing a guid is a way to guarantee that no matter what you do -- whether you move the fragment to another place or rename it -- it will be stable with regards to the user's customizations.

When the fragment is modifying an existing Terminal profile that came from another source (came with Terminal, detected via the Windows Subsystem for Linux detector, etc.) that was not the user¹, it should contain an updates guid. If the profile it is "updating" does not exist, the fragment will be ignored.

¹ This is so that a fragment cannot manipulate the user's personal profiles, no matter how hard it tries.

@DHowett
DHowett commented 13 days ago
In a diagram (which I'll formalize), it's like this:

-
|+ 0 - THE BEGINNING OF TIME
|
|+ 1 - The oceans form
|
|+ 2 - Built-in Terminal profiles (PowerShell, Command Prompt) are loaded
|+ 3 - Dynamic Terminal profiles (PowerShell 6+, WSL) are loaded
|
|+ 4 - Fragments are loaded
|      Fragments with a `guid` become full-fledged profiles
|      Fragments with an `updates` are applied to profiles in [1], [2], [3]
|         If they do not match, they are deleted.
|
|+ 5 - User profiles are loaded
|      Profiles that have the same GUID as ones in 1,2,3,4 are overlaid on
|      top as user-specific customizations.
|      If the user has a profile that came from [3] or [4] that can no longer
|      be found, it is deleted (So, if they uninstall Git for Windows the
|      profile doesn't linger)
v
