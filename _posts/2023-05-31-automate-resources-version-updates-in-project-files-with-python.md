---
layout: post
title: Automate resources version updates in project files with Python
categories: Python automation
tags: [Python, automation, resource version control]
author: denzza
---


![python-automation](/images/2023-05-31-automate-resources-version-updates-in-project-files-with-python/PythonAutomation.png)
*Image credit: [RealPython](https://realpython.com/python-web-applications/)*

Keeping track of and managing resources versions and version numbers can be a tedious task, especially when working on large projects with many files. It becomes even more cumbersome when these version numbers are embedded within the code across different files. However, automation with Python can come to the rescue.
Working with countless integrations myself that on regular basis needs a version change in WSDL or XSD resources because of the new element introduced or any type of change where is good to have a new version number of the service resource used, this post maybe useful for you. 

Today, I'll be introducing a simple Python script that helps automate the process of updating version strings and namespaces in a project's files. This script is useful in scenarios where a project has multiple files that reference a particular version string or a namespace that needs to be updated regularly, for instance, in Web Services Description Language (WSDL) and XML Schema Definition (XSD) resources.
This I have done manually:
- Create a new resource file version **Service_v1.wsdl** becomes **Service_v2.xsdl** that will be used in the project moving forward. In addition .XSD resources follow these.
- Find all the files that in wich namespace needs to be updated
- Open each file manually and do a find/repalce
- Check again that all the correct files have been updated
- Most often then not my build or deployment of the project feils because of something I missed. Big human error factor here. Go back find and replace to the correct namespace. 

This is exaclty what the script does automaticaly.
It works by walking through each file in a specified project directory, checking for an old version string/namespace, and replacing it with a new version string/namespace. It also has the capability to exclude specific files from this operation, which is sometimes necessary when dealing with specific files that should not be altered.

### How to Use the Script ###

Here are the configuration steps to use the script (Dependencies: Python 3):

1. Update the configRun.py script with the namespaces that needs to be found and replaced. The old_version : new_version pairs represent the strings you want to find and replace in your project files. You can add as many pairs as you want.
2. Set the project directory to correct path
3. Backup the project before you run the script. - If needed, but you can always just download the latest version from version control
4. Set the files that you want to exclude from changes

### Executing program ###
1. Navigate to the location of the script, exmpl: 
```
cd C:\dev\GIT\Tools\versioncange
```

2. run the configRun.py script in windows cmd:
```
python configRun.py
```

### The Code ###

Here is the code for configRun.py: 
```
from versionChange import replace_versions

#############  CONFIG  ###########################################################
replacements = {
    'OLD_VERSION_1': 'NEW_VERSION_1',
    'OLD_VERSION_2': 'NEW_VERSION_2',
    # Add more old_version:new_version pairs as needed
}
# These are the files to be excluded from replacement
excluded_files = ['excluded_file_1.xsd', 'excluded_file_2.wsdl']

#exampel of project directory: 'C:/Application/Code/project_name' 
directory = '/path/to/your/project'

#############  END CONFIG  ########################################################

report, excluded_report = replace_versions(directory, replacements, excluded_files)

if not report:
    print("No replacements made in the project files")
else:
    for filepath, num_replacements in report.items():
        print(f'Replaced strings in {filepath} ({num_replacements} replacements)')

print("\nFiles excluded from replacement:")
for filepath in excluded_report:
    print(filepath)
```

Here is the replace_versions function where all the logic happens:

```
import os
import re

def replace_versions(directory, replacements, excluded_files):
    report = {}
    excluded_report = []

    # Walk through each file in the directory
    for root, dirs, files in os.walk(directory):
        # Ignore '.git' directories
        dirs[:] = [d for d in dirs if d != '.git']
        for filename in files:
            filepath = os.path.join(root, filename)
            if filename in excluded_files:
                excluded_report.append(filepath)
                continue

            with open(filepath, 'r', encoding='ISO-8859-1') as file:
                filedata = file.read()

            # Use regular expressions to find the old versions and replace them
            for old_version, new_version in replacements.items():
                filedata, num_replacements = re.subn(old_version, new_version, filedata)
                if num_replacements > 0:
                    report[filepath] = report.get(filepath, 0) + num_replacements

            # If replacements were made, update the file
            with open(filepath, 'w', encoding='UTF-8') as file:
                file.write(filedata)

    return report, excluded_report
```
 

The script reads the configuration from the configRun.py file, performs the find/replace operations and prints a report indicating where replacements were made and how many replacements were made per file. If no replacements were made, it outputs the message "No replacements made in the project files".

### The Power of Automation ###
The advantage of using a script like this is the time saved from manually searching and replacing version numbers in a large number of files. It also minimizes the potential for human error that could result from manual replacement.

Furthermore, the script has built-in flexibility. For example, you can easily update the strings to replace or add new ones by simply updating the configRun.py script. Also, you can specify files to be excluded from the operation, providing granular control over which files are affected by the version updates.

In conclusion, automating version updates in project files is a great example of how Python can be leveraged to perform routine tasks more efficiently and accurately. This not only saves precious development time but also helps maintain a more robust codebase. If you frequently find yourself making manual version updates across multiple files, give this script a try!