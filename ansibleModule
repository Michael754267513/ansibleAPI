# -*- coding: utf-8 -*-
import json
import os
from collections import namedtuple
from ansible.parsing.dataloader import DataLoader
from ansible.vars.manager import VariableManager
from ansible.inventory.manager import InventoryManager
from ansible.playbook.play import Play
from ansible.executor.task_queue_manager import TaskQueueManager
from ansible.plugins.callback import CallbackBase
from tempfile import NamedTemporaryFile
from ansible.executor.playbook_executor import PlaybookExecutor


class ResultCallback(CallbackBase):

    def __init__(self):
        self.result_json = []
        self.run_host_ok = []
        self.run_host_failed = []
        self.run_host_unreachable = []

    def v2_runner_on_ok(self, result, **kwargs):
        host = result._host
        self.result_json.append(json.dumps({host.name: result._result}, indent=4))
        self.run_host_ok.append(host.name)

    def v2_runner_on_failed(self, result, **kwargs):
        host = result._host
        self.result_json.append(json.dumps({host.name: result._result}, indent=4))
        self.run_host_failed.append(host.name)

    def v2_runner_on_unreachable(self, result,**kwargs):
        host = result._host
        self.result_json.append(json.dumps({host.name: result._result}, indent=4))
        self.run_host_unreachable.append(host.name)

    def v2_playbook_on_start(self, playbook):
        self.playbook_on_start()

    def v2_playbook_on_notify(self, handler, host):
        self.playbook_on_notify(host, handler)

    def v2_playbook_on_no_hosts_matched(self):
        self.playbook_on_no_hosts_matched()

    def v2_playbook_on_no_hosts_remaining(self):
        self.playbook_on_no_hosts_remaining()

    def v2_playbook_on_task_start(self, task, is_conditional):
        self.playbook_on_task_start(task.name, is_conditional)

    def v2_playbook_on_cleanup_task_start(self, task):
        pass  # no v1 correspondence

    def v2_playbook_on_handler_task_start(self, task):
        pass  # no v1 correspondence

    def v2_playbook_on_vars_prompt(self, varname, private=True, prompt=None, encrypt=None, confirm=False, salt_size=None, salt=None, default=None):
        var = self.playbook_on_vars_prompt(varname, private, prompt, encrypt, confirm, salt_size, salt, default)

    def v2_playbook_on_import_for_host(self, result, imported_file):
        host = result._host.get_name()
        self.playbook_on_import_for_host(host, imported_file)

    def v2_playbook_on_not_import_for_host(self, result, missing_file):
        host = result._host.get_name()
        self.playbook_on_not_import_for_host(host, missing_file)

    def v2_playbook_on_play_start(self, play):
        self.playbook_on_play_start(play.name)

    def v2_playbook_on_stats(self, stats):
        self.playbook_on_stats(stats)

    def v2_on_file_diff(self, result):
        print(result._result)
        if 'diff' in result._result:
            host = result._host.get_name()
            self.on_file_diff(host, result._result['diff'])


class AnsibleBase(object):

    def __init__(self, targetHost):
        self.Options = namedtuple('Options',
                             ['connection', 'module_path', 'forks', 'become', 'become_method', 'become_user', 'check',
                              'diff', 'listhosts', 'listtasks', 'listtags', 'syntax', 'tags', 'skip_tags'])

        # initialize needed objects
        self.variable_manager = VariableManager()
        self.options = self.Options(connection='smart',
                               module_path=['/usr/lib/python2.7/site-packages/ansible/modules'],
                               forks=10,
                               become=False,
                               become_method='sudo',
                               become_user='root',
                               check=False,
                               diff=False,
                               listhosts=False,
                               listtasks=False,
                               listtags=False,
                               syntax=False,
                               tags=[],
                               skip_tags=[],
                               )

        self.loader = DataLoader()  # Takes care of finding and reading yaml, json and ini files
        self.passwords = dict(vault_pass='secret')
        self.results_callback = ResultCallback()
        self.hostsFile = NamedTemporaryFile(delete=False)
        # 指定用户名密码
        ansible_ssh_user = 'username'
        ansible_ssh_pass = 'password'
        for host in targetHost:
            self.hostsFile.write('%s ansible_ssh_user=%s ansible_ssh_pass=%s\r\n' % (host,ansible_ssh_user,ansible_ssh_pass))
        # self.hostsFile.write(targetHost)
        self.hostsFile.close()
        self.inventory = InventoryManager(loader=self.loader, sources=self.hostsFile.name)
        # self.inventory = InventoryManager(loader=self.loader, sources='/etc/ansible/hosts')
        self.variable_manager = VariableManager(loader=self.loader, inventory=self.inventory)

    def ansiblePlayModule(self, module, args, become=False, become_method='sudo', become_user='root'):
        # create play with tasks
        play_source = dict(
            name='Ansible Task',
            hosts='all',
            gather_facts='no',
            become=become,
            become_method=become_method,
            become_user=become_user,
            tasks=[
                dict(action=dict(module=module, args=args), register='shell_out'),
            ]
        )
        play = Play().load(play_source, variable_manager=self.variable_manager, loader=self.loader,)
        # Run it - instantiate task queue manager, which takes care of forking and setting up all objects to iterate over host list and tasks
        tqm = None
        # run it
        try:
            tqm = TaskQueueManager(
                inventory=self.inventory,
                variable_manager=self.variable_manager,
                loader=self.loader,
                options=self.options,
                passwords=self.passwords,
                stdout_callback=self.results_callback, #回调显示结果
            )
            result = tqm.run(play)
        finally:
            if tqm is not None:
                tqm.cleanup()
                # os.remove(self.hostsFile.name)
                self.inventory.clear_pattern_cache()
                self.remove_tempfile()

    def ansiblePlayBook(self, playbook_path, tags=[], skip_tags=[], forks=10, become_user='root', become_method='sudo', extra_vars={},become=False):
        self.options = self.Options(connection='smart',
                                    module_path=['/usr/lib/python2.7/site-packages/ansible/modules'],
                                    forks=forks,
                                    become=become,
                                    become_method=become_method,
                                    become_user=become_user,
                                    check=False,
                                    diff=False,
                                    listhosts=False,
                                    listtasks=False,
                                    listtags=False,
                                    syntax=False,
                                    tags=tags,
                                    skip_tags=skip_tags,
                                    )
        self.variable_manager.extra_vars = extra_vars
        pb = None
        try:
            pb = PlaybookExecutor(
                playbooks=playbook_path,
                inventory=self.inventory,
                variable_manager=self.variable_manager,
                loader=self.loader,
                options=self.options,
                passwords=self.passwords,
            )
            # pb._tqm._stdout_callback = self.results_callback
            result = pb.run()
        finally:
            self.remove_tempfile()

    def remove_tempfile(self):
        os.remove(self.hostsFile.name)

    def get_result(self):
        # 获取结束回调
        self.result_all = {'success': {}, 'fail': {}, 'unreachable': {}}
        return self.results_callback.result_json

    def get_json(self):
        d = self.get_result()
        data = json.loads(d)
        return data
