#

ip_list = ["10.148.181.52"]
#extra_vars = {"cluster": "true", "es_node":"%s"%ip_list,  "proles": ["elasticsearch"]}  # es
extra_vars = {'proles':'jdk1.6'}  
playbook = ["/root/eman/ansible/install_playbook.yml"]
ansible_test = AnsibleBase(targetHost=ip_list)
status = ansible_test.ansiblePlayBook(extra_vars=extra_vars,playbook_path=playbook, become=True)
# ansible_test.ansiblePlayBook(extra_vars=extra_vars, playbook_path=playbook, tags=tags, skip_tags=skip_tags)  #跳过tags 里面包含skip_tags
# ansible_test.ansiblePlayBook(extra_vars=extra_vars, playbook_path=playbook, tags=tags)
# ansible_test.ansiblePlayModule(module="shell", args="whoami",become=True, become_method='sudo',become_user='root')
print(status)
res = ansible_test.get_result()
# res = ansible_test.get_json()
for item in res:
    print(item)
