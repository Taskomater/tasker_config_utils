# tasker_config_utils

This is a project that provides bash util script(s) to extract information and modify [Tasker App] XML files. Check [Tasker XML Info] for more info on how Tasker App XML files work.
##


### Contents
- [Compatibility](#Compatibility)
- [Dependencies](#Dependencies)
- [Downloads](#Downloads)
- [Install Instructions For Termux In Android](#Install-Instructions-For-Termux-In-Android)
- [tasker_config_utils Script Usage](#tasker_config_utils-Script-Usage)
- [Current Features](#Current-Features)
- [Planned Features](#Planned-Features)
- [Issues](#Issues)
- [Worthy Of Note](#Worthy-Of-Note)
- [FAQs And FUQs](#FAQs-And-FUQs)
- [Changelog](#Changelog)
- [Contributions](#Contributions)
##


### Compatibility

- Android using [Termux App].
- Linux distros.
- Windows using [cygwin].
##


### Dependencies

- Android users should install [Termux App].
- Windows users should install [cygwin] and use cygwin shell to run scripts.
- `GNU` variant of `sed` should be installed in your system.
  - Termux (non-root shell): `apt install sed`
  - Linux distros: `sudo apt install sed`
  - Windows: It should ideally be installed by default with cygwin. If not, then run cygwin installer and search package and install.
##


### Downloads

- [GitHub releases](https://github.com/Taskomater/tasker_config_utils/releases).
##


### Install Instructions For Termux In Android

The `tasker_config_utils` file should be placed in termux `bin` directory `/data/data/com.termux/files/usr/bin` and it should have `termux` `uid:gid` ownership and have executable `700` permission before it can be run in the termux terminal without specifying its path.
1. Copy the file to termux bin directory:
  Either `cd` to the download/extraction directory and run following commands

  ```
  cat tasker_config_utils > /data/data/com.termux/files/usr/bin/tasker_config_utils
  ```

  Or use a file browser like root explorer to copy the file to the termux bin directory.

2. Set correct ownership and permission:
  Either run following commands to set them automatically, requires su binary to be in `$PATH`.

  ```
  export termux_bin_path="/data/data/com.termux/files/usr/bin"; export owner="$(stat -c "%u" "$termux_bin_path")"; for f in tasker_config_utils; do if [ -f "$termux_bin_path/$f" ]; then su -c "chown $owner:$owner \"$termux_bin_path/$f\" && chmod 700 \"$termux_bin_path/$f\""; fi; done;
  ```

  Or manually set them with your file browser. You can find `termux` `uid` and `gid` by running the command `id -u` in a non root shell in termux or by checking the properties of the termux `bin` directory from your file browser.
##


### tasker_config_utils Script Usage

You may optionally run the script placed in the current directory without setting the ownership and permission by running the command `bash tasker_config_utils`.
If you are not running the script in termux, set the shebang of the script correctly (the first line of the script).

Any full exported tasker config passed to the script must have been generated using the `Data->Backup` Tasker menu option or the `Data Backup` action. The script will not work with `autobackups` since those contain XML nodes that are collapsed to a single line with no indentations. If you have an `autobackup`, then import that into Tasker and export a backup manually again with one of the ways mentioned earlier. This does not apply for exported project, task, profile or scene files.

- **`tasker_config_utils`**

##### Help:

```
Usage:
  tasker_config_utils [ -h | --help ]
  tasker_config_utils [ --version ]
  tasker_config_utils command [command_options]

Available commands:
  extract_tag           extract tags from tasker config XML file
  convert_project       convert project into a non-standalone project
  generate_config_info  generate a markdown config info file for a given tasker config XML file


Use "tasker_config_utils command [ -h | --help ]" for more help about a command.
```
##


- **`tasker_config_utils extract_tag`**

**`tasker_config_utils extract_tag`** command is used to extract tags of nodes from a tasker config file.

##### Help:

```
tasker_config_utils extract_tag command is used to extract tags of nodes from a tasker config XML file.


Usage:
  tasker_config_utils extract_tag [command_options] tasker_config

Available command_options:
  [ -h | --help ]    display this help screen
  [ -v | -vv ]       set verbose level to 1 or 2
  [ -a ]             extract Project node tag
  [ -e ]             extract tags from entire node
  [ -p ]             extract Profile node tag
  [ -s ]             extract Task node tag
  [ -t ]             extract Scene node tag
  [ --node=<node> ]
                     optional node to extract
  [ --tag=<tag> ]
                     optional tag to extract
  [ --pre_tag=<tag> ]
                     optional pre tag sed regex to match while extracting tag
  [ --post_tag=<tag> ]
                     optional post tag sed regex to match while extracting tag

tasker_config should be the path to a Tasker "Data Backup" XML file. It passed must be an exported \
"Data Backup" and not an auto backup.

The options '-a', '-p', '-s', '-t' and '--node' set the extraction_node of the extract_tag command. \
One of them must be passed.

For the option '-a', the default tag is 'name'. For all others, the default tag is 'nme'.

By default tags will only be extracted from the node itself and not any of its sub nodes. Multi-line tags \
can be extracted. However, this only allows the first tag of the same type to be extracted from each node. 

The '-e' option can be passed to extract tags from the node and its sub nodes. Only single line tags can \
be extracted. However, this allows multiple tags of the same type to be extracted from each node.

set verbose level to 1 or 2 to get more info when running tasker_config_utils extract_tag command.
```

##### Examples:

```
#extract all profile names
tasker_config_utils extract_tag -p "config.xml"

#extract all scene names
tasker_config_utils extract_tag -s "config.xml"

#extract all task names
tasker_config_utils extract_tag -t "config.xml"

#extract all project names
tasker_config_utils extract_tag -a "config.xml" 

#extract all task names with "Run Both Together" collision handling
tasker_config_utils extract_tag -t --post_tag='<rty>2<\/rty>' "config.xml"

#extract all scene names from the project "Base"
tasker_config_utils extract_tag -a --pre_tag='<name>Base<\/name>' --tag=scenes "config.xml" 

#extract all profile, scene and task names from tasker_config
tasker_config="config.xml"; echo -e "Profiles:"; tasker_config_utils extract_tag -p "$tasker_config"; echo -e "\nScenes:"; tasker_config_utils extract_tag -s "$tasker_config"; echo -e "\nTasks:"; tasker_config_utils extract_tag -t "$tasker_config";

#extract all task labels, the piped sed commands convert XML special characters to ascii
tasker_config_utils extract_tag --node='Action' --tag='label' "config.xml" | sed -e 's/&amp;/\&/g' -e 's/&lt;/</g' -e 's/&gt;/>/g'

#extract all task anchor labels, the piped sed commands convert XML special characters to ascii
tasker_config_utils extract_tag --node='Action' --pre_tag='<code>300<\/code>' --tag='label' "config.xml" | sed -e 's/&amp;/\&/g' -e 's/&lt;/</g' -e 's/&gt;/>/g'

#extract all task help anchor(action 0) labels of tasks, the piped sed commands convert XML special characters to ascii
tasker_config_utils extract_tag --node='Action' --pre_tag='sr="act0".*<code>300<\/code>' --tag='label' "config.xml" | sed -e 's/&amp;/\&/g' -e 's/&lt;/</g' -e 's/&gt;/>/g'

#extract all scene tap events anonymous task ids
tasker_config_utils extract_tag -s -e --tag='itemclickTask' "config.xml" 

#extract all scene anonymous task ids
tasker_config_utils extract_tag -s -e --tag='[a-zA-Z0-9]+Task' "config.xml"

#extract profile id of profile using entry task with task id 666 and also print the matching profile nodes
tasker_config_utils extract_tag -p -e --tag="id" --post_tag='<mid0>666<\/mid0>' -vv "config.xml" 
```

If you are passing `--tag`, `--pre_tag` or `--post_tag` options with the `-e` option, it might be helpful to pass `-vv` for verbose mode so that the matching nodes can also be printed on the console in addition to the extracted tag values.
##


- **`tasker_config_utils convert_project`**

**`tasker_config_utils convert_project`** command is used to convert a project into a non-standalone project so that it does not have profiles, scenes and tasks that were originally not in the project in the tasker_config. The details on how this works is in the function header of the script.

The reason why this is needed is because of some issues in the export `Project` XML design. The problem is that say you have 3 projects. project 1 and 2 use common helper profiles, tasks and scenes which are all defined in project 3. Now if you export project 1 and 2 and share it with someone. They will only be able to import project 1 and importing project 2 will fail because when they imported project 1 some or all the helper tasks from project 3 were also imported, now they can't import project 2 because Tasker config already contains tasks with the same name.

Moreover when project 1 is imported, the tasks from project 1 and 3 are mixed inside the same project. This creates a mess and destroys the purpose of separate projects.

There should be a way to export projects with only tasks, scenes and profiles referenced inside the same project only. This way each project 1, 2 and 3 can be exported separately and imported separately so that there aren't any conflicts when importing. The temporary missing references would not be that big of a problem as long as the projects containing the missing tasks, scenes and profiles are also imported next. 

Moreover, the order of the profiles and tasks in their respective tabs is not saved when sharing projects, at least currently. The order and id of profiles and tasks belonging to a project is saved in its `Project` node of the Tasker config file in the `<pids>` and `<tids>` tags respectively. When you export a `Project` XML file, the order is the same as the one in the Tasker config file. But when you import the XML file into a different device, the ids are changed for all profiles and tasks so that there is not any conflict with the ids already existing in the Tasker config of the device. Ideally new ids should be assigned in the same order as they exist in the `Project` XML file but there are not currently resulting in loss of sort order.

It might be helpful if you upvoted the [Tasker Feature Request](https://tasker.helprace.com/i459-project-export-without-including-profiles-tasks-scenes-from-other-projects) for these issues if you think this is something you would like to have so that support for this can be added directly in Tasker and this potentially unsafe XML modification is not needed. João is already aware of this but its not a high priority feature for him currently.

##### Help:

```
tasker_config_utils convert_project command is used to convert a project into a non-standalone project \
so that it does not have profiles, scenes and tasks that were originally not in the project in the \
tasker_config. tasker_config must be an exported data backup and not an auto backup.


Usage:
  tasker_config_utils convert_project [command_options] tasker_config current_exported_tasker_project new_exported_tasker_project project_name

Available command_options:
  [ -h | --help ]    display this help screen
  [ -v | -vv ]       set verbose level to 1 or 2


tasker_config should be the path to a Tasker "Data Backup" XML file. It passed must be an exported \
"Data Backup" and not an auto backup.

current_exported_tasker_project should be the path to a Tasker exported "Project" XML file that needs to \
be converted. It must have been exported from Tasker with a config that is the same as the tasker_config file.

new_exported_tasker_project should be the path to the output "Project" XML file.

project_name should be the Tasker project name which was exported to current_exported_tasker_project. Its \
project node must exist inside both the tasker_config and current_exported_tasker_project files. Its \
Project node in the current_exported_tasker_project should also have a 'sr' value of 'proj0'.


Example Usage:
Create a Tasker "Data Backup". Example: "config.xml".
Export Tasker "Foo Bar" Project. It should by default be exported to "Foo_Bar.prf.xml".
Place both files in same folder and run following command:
  tasker_config_utils convert_project -v "config.xml" "Foo_Bar.prf.xml" "Foo_Bar-out.prf.xml" "Foo Bar"

Optionally use the "Convert Tasker Project File To Non-Standalone Project File" task from the "Tasker Config Utils" project.

set verbose level to 1 or 2 to get more info when running tasker_config_utils convert_project command.
```

##### Examples:

```
#convert a project file into a non-standalone project file
tasker_config_utils convert_project -v "config.xml" "Foo_Bar.prf.xml" "Foo_Bar-out.prf.xml" "Foo Bar" 
```
##


- **`tasker_config_utils generate_config_info`**

**`tasker_config_utils generate_config_info`** command is used to generate a markdown config info file for a given Tasker XML file.

Generated config info file will contain:
1) *Export info like tasker version and timestamp.*
2) *Names of all profiles, scenes and tasks in the config or project.*
3) *Profiles info like their settings, entry and exit tasks, etc.*
4) *Tasks info like their settings, the label of the first action of the task if its an anchor action as help, etc.*

More info will be added later.

##### Help:

```
tasker_config_utils generate_config_info command is used to generate a markdown config info file for a given tasker config XML file.

Generated config info file will contain:
1) Export info like tasker version and timestamp.
2) Names of all profiles, scenes and tasks in the project.
3) Profiles Info like their settings, entry and exit tasks, etc.
4) Tasks Info like their settings, the label of the first action of the task if its an anchor action as help, etc.

Usage:
  tasker_config_utils generate_config_info [command_options] -a exported_tasker_config exported_tasker_config_info
  tasker_config_utils generate_config_info [command_options] -p exported_tasker_config exported_tasker_config_info project_name

Available command_options:
  [ -h | --help ]    display this help screen
  [ -v | -vv ]       set verbose level to 1 or 2
  [ -a ]             extract all info
  [ -p ]             extract info of a specific project

The options '-a' and '-p' set the generate_config_info_mode of the generate_config_info command. One of \
them must be passed. The

The '-a' option sets the generate_config_info_mode to "all" mode. If this is passed, \
then exported_tasker_config can be any type of exported Tasker XML file like a "Data Backup", 
"Profile", "Project", "Task" or a "Scene" XML file of which config info needs to be generated. \
The info of all profiles, scenes and tasks inside the file will be generated. This is likely \
to take some time depending on XML file size.

The '-p' option sets the generate_config_info_mode to "project" mode. If this is passed, then \
exported_tasker_config should be the path to a Tasker exported "Project" XML file of which the \
project info needs to be generated. You may optionally pass a Tasker "Data Backup" XML file instead. \
Only the info of profiles, scenes and tasks belonging to the project will be generated. The project_name \
should be the Tasker project name which was exported to current_exported_tasker_config. Its \
project node must exist inside the exported_tasker_config file.

exported_tasker_config_info should be the path to the output project info markdown file.


Example Usage 1:
Create a Tasker "Data Backup". Example: "config.xml".
Run following command:
  tasker_config_utils generate_config_info -v -a "config.xml" "config.md"

Example Usage 2:
Export Tasker "Foo Bar" Project. It should by default be exported to "Foo_Bar.prf.xml".
Run following command:
  tasker_config_utils generate_config_info -v -p "Foo_Bar.prf.xml" "Foo_Bar-out.prf.md" "Foo Bar"

Optionally use the "Convert Tasker Project File To Non-Standalone Project File" task from the "Tasker Config Utils" project.

set verbose level to 1 or 2 to get more info when running tasker_config_utils generate_config_info command.
```

##### Examples:

```
#generate a markdown project info file for a given tasker config file
#tasker_config_utils generate_config_info -v -a "config.xml" "config.md"

#generate a markdown project info file of a specific project
#tasker_config_utils generate_config_info -v -p "Foo_Bar.prf.xml" "Foo_Bar-out.prf.md" "Foo Bar"
```
##


### Current Features

- Extract tags and nodes from tasker config XML files.
- Convert a project into a non-standalone project.
- Generate a markdown config info file for a given tasker config XML file.
##


### Planned Features

- Add more info to the generated markdown config info file.
- Sort profiles, scenes and task nodes.
##


### Issues

`-`
##


### Worthy Of Note

`-`
##


### FAQs And FUQs

Check [FAQs_And_FUQs.md](FAQs_And_FUQs.md) file for the **Frequently Asked Questions(FAQs)** and **Frequently Unasked Questions(FUQs)**.
##


### Changelog

Check [CHANGELOG.md](CHANGELOG.md) file for the **Changelog**.
##


### Contributions

`-`
##


[Tasker App]: https://play.google.com/store/apps/details?id=net.dinglisch.android.taskerm
[Tasker XML Info]: https://github.com/Taskomater/Tasker-XML-Info
[Termux App]: https://github.com/termux/termux-app
[cygwin]: https://cygwin.com/index.html
