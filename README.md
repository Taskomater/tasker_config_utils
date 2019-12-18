# Tasker Config Utils

This is a project to provide guidance on how Tasker App XML files work and provides a script with some utils to extract information and modify Tasker App XML files.

### Downloads
Download latest release from [here](https://github.com/Taskomater/Tasker-Config-Utils/releases).
##


### **`tasker_config_utils script:`**

You can run the script placed in the current directory without setting the ownership and permission by running the command `bash tasker_config_utils`.
If you are not running the script in termux, set the shebang of the script correctly (the first line of the script).

##### Usage:

```
Usage:
  tasker_config_utils [ -h | --help ]
  tasker_config_utils [ --version ]
  tasker_config_utils command [command_options]

Available commands:
  extract_tag      extract tags from tasker config
  convert_project  convert project into a non-standalone project


Use "tasker_config_utils command [ -h | --help ]" for more help about a command.
```
##


- **`tasker_config_utils extract_tag`**

##### Usage:

**`tasker_config_utils extract_tag`** command is used to extract tags of nodes from a tasker config file.

```
tasker_config_utils extract_tag command is used to extract tags of nodes from a tasker config file.


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

Moreover, the order of the profiles and tasks in their respective tabs is not saved when sharing projects, atleast currently. The order and id of profiles and tasks belonging to a project is saved in its `Project` node of the Tasker config file in the `<pids>` and `<tids>` tags respectively. When you export a `Project` XML file, the order is the same as the one in the Tasker config file. But when you import the XML file into a different device, the ids are changed for all profiles and tasks so that there is not any conflict with the ids already existing in the Tasker config of the device. Ideally new ids should be assigned in the same order as they exist in the `Project` XML file but there are not currently resulting in loss of sort order.

It might be helpful if you upvoted the [Tasker Feature Request](https://tasker.helprace.com/i459-project-export-without-including-profiles-tasks-scenes-from-other-projects) for these issues if you think this is something you would like to have so that support for this can be added directly in Tasker and this potentially unsafe XML modification is not needed. João is already aware of this but its not a high priority feature for him currently.

##### Usage:

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

##### Usage:

```
tasker_config_utils generate_config_info command is used to generate a markdown config info file for a given Tasker XML file.

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


### Dependencies:

You should have GNU sed installed in your system.
- Termux (non-root shell): `apt install sed`
- Linux distros: `sudo apt install sed`
- Windows: You can use cygwin installer.
##


### Install Instructions For Termux In Android:

The `tasker_config_utils` file should be placed in termux `bin` directory `/data/data/com.termux/files/usr/bin` and it should have `termux` uid:gid ownership and have executable `700` permission before it can be run in the termux terminal without specifying its path.
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

You may optionally run the script placed in the current directory without setting the ownership and permission by running the command `bash tasker_config_utils`.
##




## Tasker XML Export Types:

Tasker supports 4 types of XML exports.

- **`Data Backup`**
	It contains the entire tasker config with all profiles, scenes and tasks.  
	*XML file suffix:* `.xml`  
	*Import Instructions:* `Tasker Options In Tasker Home Screen` -> `Data` -> `Restore` -> `User Local Backup`  
	*Export Instructions 1:* `Tasker Options In Tasker Home Screen` -> `Data` -> `Backup`  
	*Export Instructions 2:* Tasker Task Action  `Tasker` -> `Data Backup`  

- **`Project`**
	It contains all profiles, scenes and tasks that are referenced in the project that is exported from the tasker config. Basically if you are exporting project 1, then the exported XML file will contain all profiles, scenes and tasks that are inside project 1 in the tasker config. Additionally, it will also contains profiles, scenes and tasks from projects other than project 1 that are referred or called by project 1 in some way. This automatically creates a standalone project file and does not require the users to export/import multiple projects. Tasker exports everything that is required for a user to run the project.  
	*XML file suffix:* `.prj.xml`  
	*Import Instructions:* `Long Press Anywhere On Bottom Nav Bar` -> `Import Project`  
	*Export Instructions:* `Long Press On The <Project Name> Tab In Bottom Nav Bar` -> `Export` -> `XML to Storage`  

- **`Profile`**
	It contains the profile and all tasks that are referenced in the profile that is exported from the tasker config. This will also export anonymous tasks of the profile that are not named and contain only an id in the tasker config, like for example the entry and exit tasks which were created without naming them. Note the `Optional` when you select `New Task` will creating a profile.  
	*XML file suffix:* `.prf.xml`  
	*Import Instructions:* `Long Press PROFILES Tab In Top Nav Bar` -> `Import Profile`  
	*Export Instructions:* `Long Press On The <Profile Name> In the PROFILES Tab` -> `Tasker Options` -> `Export` -> `XML to Storage`  

- **`Scene`**
	It contains the scene and all tasks that are referenced in the scene that is exported from the tasker config. This will also export anonymous tasks of the scene and sub scenes that are not named and contain only an id in the tasker config. The key and element tasks of the scenes are anonymous tasks.  
	*XML file suffix:* `.scn.xml`  
	*Import Instructions:* `Long Press SCENES Tab In Top Nav Bar` -> `Import One Scene`  
	*Export Instructions:* `Long Press On The <Scene Name> In the SCENES Tab` -> `Tasker Options` -> `Export` -> `XML to Storage`  

- **`Task`**
	It contains only the task that is exported from the tasker config.  
	*XML file suffix:* `.tsk.xml`  
	*Import Instructions:* `Long Press TASKS Tab In Top Nav Bar` -> `Import Task`  
	*Export Instructions:* `Long Press On The <Task Name> In the TASKS Tab` -> `Tasker Options` -> `Export` -> `XML to Storage`  

If the profile, scene or task XML file does not have the correct suffix, it will not be displayed in the import menu by Tasker.

The initial project's name is `Base` and it cannot be exported. Create a new project and move required profiles, scenes and tasks to it and then export that. 

If your tasker config contains a named profile, scene or task and you are trying to import a XML file that contains a profile, scene or task with the same name, then the import will fail. 
##


## Tasker XML File Structure:

The exported files depending on the export type will have the following XML node structure:
```
<TaskerData sr="" dvi="1" tv="5.9.rc">
	<dmetric>1440.0,2392.0</dmetric>
	<Profile sr="prof1" ve="2">
		...
		<id>1</id>
		...
	</Profile>
	<Profile sr="prof2" ve="2">
		...
		<id>2</id>
		...
	</Profile>
	....
	<Project sr="proj0" ve="2">
		...
		<name>Base</name>
		<pids>1,2</pids>
		<scenes>Scene 1,Scene 2</scenes>
		<tids>1,2</tids>
	</Project>
	...
	<Scene sr="sceneScene 1">
		...
		<nme>Scene 1</nme>
		...
	</Scene>
	<Scene sr="sceneScene 2">
		...
		<nme>Scene 2</nme>
		...
	</Scene>
	...
	</Task>
		<Task sr="task1">
		...
		<id>1</id>
		...
	</Task>
	</Task>
		<Task sr="task2">
		...
		<id>2</id>
		...
	</Task>
	...
</TaskerData>
```

To find what type of exported file a given XML file is, even if it is not correctly named or described, examine the nodes in the XML file.
```
- If there is only one `Task` node; then
	It's a Task file.
- If there is only one `Scene` node and some optional `Task` nodes without `<nme>` tags; then
	It's a Scene file.
- If there is only one `Profile` node and some `Task` nodes; then
	It's a Project file.
- If the there is only one `Project` node and its `<name>` tag does not contain `Base`; then
	It's a Project file.
- Else
	It's a Data Backup file.
```
##


## Tasker XML File Nodes And Tags:

The `Profile`, `Project`, `Scene` and `Task` nodes are optional depending on the export type. Atleast one needs to exist.

- **`dmetric Tag:`**

	The `<dmetric>` tag is an optional tag depending on if the XML contains a `Scene` node. It stands for `display metrics`. The tag contains the width and height of the display that is usable by an App. The height value will be the real height of the display minus the height of the navigation bar. The values will depend on which device the XML file was exported from so that the required scaling of scenes can be done when the XML file is imported into a Tasker on a device with different display metrics.
##


- **`Profile Node:`**

	*Required Root Tags:*
	
	- `<id>` defines the unique id of the profile.

	*Optional Root Tags:*
	- `<mid0>` defines the entry task id of the profile.
	- `<mid1>` defines the exit task id of the profile.
	- `<nme>` defines the unique name of the profile.

	The `<mid0>` and `<mid1>` tags will only exist if a task is actually set as the profile entry or exit task. Atleast one needs to be defined. If either tags exists and the task id in the tag is not referencing a named task, then it must be referencing an anonymous task.

	*Optional Sub Nodes:*
	- `App` defines the info for an Application Context.
	- `Day` defines the info for a Day Context.
	- `Event` defines the info for an Event Context.
	- `Loc` defines the info for a Location Context.
	- `State` defines the info for a State Context.
	- `Time` defines the info for a Time Context.

	Atleast one `Context` sub node needs to be defined.
##


- **`Project Node:`**

	*Required Root Tags:*
	
	- `<name>` defines the unique name of the project.	

	*Optional Root Tags:*
	- `<pids>` defines the profile ids of project.
	- `<scenes>` defines the scene names of project.
	- `<tids>` defines the task ids of project.

	The `<pids>`, `<scenes>` and `<tids>` tags will only exist if atleast one profile, scene or task exists in the project respectively. The ids of anonymous profile or scene tasks are not listed in the `<tids>` tag of exported `Data Backup` XMl files. However, they are listed in exported `Project` XML files.
##


- **`Scene Node:`**

	*Required Root Tags:*
	
	- `<nme>` defines the unique name of the scene.

	*Optional Root Tags:*

	*Required Sub Nodes:*
	- `PropertiesElement` defines the info for the `Scene Properties`.

	*Optional Sub Nodes:*
	- `ButtonElement` defines the info for a `Button` Element.
	- `CheckBoxElement` defines the info for a `Checkbox` Element.
	- `DoodleElement` defines the info for a `Doodle` Element.
	- `EditTextElement` defines the info for a `TextEdit` Element.
	- `ImageElement` defines the info for an `Image` Element.
	- `ListElement` defines the info for a `Menu` Element.
	- `OvalElement` defines the info for an `Oval` Element.
	- `PickerElement` defines the info for an `Oval` Element.
	- `RectElement` defines the info for a `Rect` Element.
	- `SceneElement` defines the info for a `Map` Element.
	- `SliderElement` defines the info for a `Slider` Element.
	- `SpinnerElement` defines the info for a `Spinner` Element.
	- `SwitchElement` defines the info for a `Switch` Element.
	- `TextElement` defines the info for a `Text` Element.
	- `ToggleElement` defines the info for a `Toggle` Element.
	- `VideoElement` defines the info for a `Video` Element.
	- `WebElement` defines the info for a `WebView` Element.

	Depending on the elements in the scene or the key event of the scene, the nodes may contain one or more tags called  `checkchangeTask`, `clickTask`, `focuschangeTask`, `itemselectedTask`, `keyTask`, `linkclickTask`, `longclickTask`, `mapclickTask`, `maplongclickTask`, `pageloadedTask`, `strokeTask`, `valueselectedTask`, `videoTask`, etc. The tags will contain the task id of the anonymous task that will be run when the respective event is triggered. The tags and anonymous task nodes will only exist if atleast one action is defined in the respective event tabs like `TAP`, `LONG TAP`, `STROKE`, etc. An anonymous task will be created regardless of if the only action is a `Perform Task` action.
##


- **`Task Node:`**

	*Required Root Tags:*
	
	- `<id>` defines the unique id of the task.

	*Optional Root Tags:*
	- `<nme>` defines the unique name of the task.
	- `<rty>` defines the `Collision Handling` option of the task. `0` or `no tags` == `Abort New Task`, `1` == `Abort Existing Task` and `2` == `Run Both Together`.
	- `<pri>` defines the priority of the task used in some cases.
	- `<stayawake>` defines the `Keep Device Awake` option of the task. `false` or `no tags` == `Do Not Keep Awake` and `true` == `Keep Awake`.

	*Required Sub Nodes:*
	- `Action` defines the info for an action.

	Atleast one `Action` sub node needs to be defined.

## Tasker Example XML:

```
<TaskerData sr="" dvi="1" tv="5.9.rc">
	<dmetric>1440.0,2392.0</dmetric>
	<Profile sr="prof10" ve="2">
		...
		<id>10</id>
		<mid0>112</mid0>
		<nme>Profile 1</nme>
		<Event sr="con0" ve="2">
			...
		</Event>
	</Profile>
	<Profile sr="prof15" ve="2">
		...
		<id>15</id>
		<mid0>150</mid0>
		<mid1>151</mid1>
		<nme>Profile 2</nme>
		<Event sr="con0" ve="2">
			...
		</Event>
		<Event sr="con1" ve="2">
			...
		</Event>
		...
	</Profile>
	<Project sr="proj0" ve="2">
		...
		<name>Some Project</name>
		<pids>10,15</pids>
		<scenes>Scene 1,Scene 2</scenes>
		<tids>112,150</tids>
	</Project>
	<Scene sr="sceneScene 1">
		...
		<nme>Scene 1</nme>
		...
		<TextElement sr="elements0" ve="3">
			...
			<itemclickTask>101</itemclickTask>
			...
		</TextElement>
		<TextElement sr="elements1" ve="3">
			...
		</TextElement>
		...
		<PropertiesElement sr="props">
			...
		</PropertiesElement>
	</Scene>
	<Scene sr="sceneScene 2">
		...
		<nme>Scene 2</nme>
		...
		<PropertiesElement sr="props">
			...
			<keyTask>103</keyTask>
			...
		</PropertiesElement>
	</Scene>
	<Task sr="task112">
		...
		<id>112</id>
		<nme>Task Called By Profile 1 As Entry Task</nme>
		...
		<Action sr="act0" ve="7">
			...
		</Action>
		<Action sr="act1" ve="7">
			...
		</Action>
		...
	</Task>
		<Task sr="task150">
		...
		<id>150</id>
		<nme>Task Called By Profile 2 As Entry Task</nme>
		...
		<Action sr="act0" ve="7">
			...
		</Action>
		<Action sr="act1" ve="7">
			...
		</Action>
		...
	</Task>
	</Task>
		<Task sr="task151">
		...
		<id>151</id>
		...
		<Action sr="act0" ve="7">
			...
		</Action>
		...
	</Task>
	</Task>
		<Task sr="task101">
		...
		<id>101</id>
		...
		<Action sr="act0" ve="7">
			...
		</Action>
		...
	</Task>
	</Task>
		<Task sr="task103">
		...
		<id>103</id>
		...
		<Action sr="act0" ve="7">
			...
		</Action>
		...
	</Task>
</TaskerData>
```

The `Profile 1` will run the named task with id `112` as entry task set in the `<mid0>` tag.
The `Profile 1` will not run any exit task as no `<mid1>` tag exists.

The `Profile 2` will run the named task with id `150` as entry task set in the `<mid0>` tag.
The `Profile 2` will run the anonymous task with id `151` as exit task set in the `<mid1>` tag.

The `TAP` event of the Text Element with `sr="elements0"` of `Scene 1` will run the anonymous task with id `101` set in the `<itemclickTask>` tag.
The `Key` event of the `Scene 2` will run the anonymous task with id `103` set in the `<keyTask>` tag.

Additional information for other sub nodes and tags will be added later.
##
