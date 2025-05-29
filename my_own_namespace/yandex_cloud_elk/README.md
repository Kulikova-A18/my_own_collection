# Ansible Collection

Коллекция my_own_namespace.yandex_cloud_elk содержит

- модуль для создания файла с указываемым содержимым в указанном месте (смотреть plugins/modules/create_file.py)
- демонстрационную использующую модуль (смотреть roles/create_file)

## модуль для создания файла с указываемым содержимым в указанном мест

```
#!/usr/bin/python

# Copyright: (c) 2018, Terry Jones <terry.jones@example.org>
# GNU General Public License v3.0+ (see COPYING or https://www.gnu.org/licenses/gpl-3.0.txt)
from __future__ import (absolute_import, division, print_function)
metaclass = type

DOCUMENTATION = r'''
---
module: create_file

short_description: This is my module for netology

version_added: "1.0.0"

description: This module creates the file in a specific path with specific content.

options:
    path:
        description: Path where the module will create the file.
        required: true
        type: str
    content:
        description: Content of the file.
        required: true
        type: str

extends_documentation_fragment:
    - netology.study.create_file

author:
    - Kulikova Alyona (@Kulikova-Alyona)
'''

EXAMPLES = r'''
# Create file in path "/test_directory/file1" with content "hello world"
- name: Test with a message
  netology.study.create_file:
    path: /test_directory/file1
    content: hello world
'''

RETURN = r'''
message:
    description: Message about the success of creating the file.
    type: str
    returned: always
    sample: 'File /test_directory/file1 was created and filled with your content'
'''

from ansible.module_utils.basic import AnsibleModule
import os


def run_module():
    module_args = dict(
        path=dict(type='str', required=True),
        content=dict(type='str', required=True)
    )

    result = dict(
        changed=False,
        message='Nothing is done'
    )

    module = AnsibleModule(
        argument_spec=module_args,
        supports_check_mode=True
    )

    if module.check_mode:
        module.exit_json(**result)

    path = module.params['path']
    content = module.params['content']

    # Create directory if it does not exist
    directory = os.path.dirname(path)
    if not os.path.exists(directory):
        try:
            os.makedirs(directory)
            result['changed'] = True
            result['message'] = f"Directory {directory} was created."
        except Exception as e:
            module.fail_json(msg=f"Failed to create directory {directory}: {str(e)}", **result)

    # Check if file exists and its content
    if os.path.isfile(path):
        with open(path, "r") as file:
            file_content = file.read()
            if file_content == content:
                module.exit_json(changed=False, message="File is already up to date")

    # Write content to the file
    try:
        with open(path, "w") as file:
            file.write(content)
        result['changed'] = True
        result['message'] = f"File {path} was created and filled with your content."
    except Exception as e:
        module.fail_json(msg=f"Failed to write to file {path}: {str(e)}", **result)

    module.exit_json(**result)


def main():
    run_module()


if __name__ == '__main__':
    main()
```
