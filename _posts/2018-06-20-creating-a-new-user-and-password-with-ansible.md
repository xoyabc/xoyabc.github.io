---
layout: post
title: 使用ansible创建新用户并设置密码
categories: linux
description: 使用ansible创建新用户并设置密码
keywords: ansible, linux
---

使用ansible的role创建用户名及随机密码，并设置密码立即过期，这样用户首次登录时需要更改为自己的密码，一是方便用户使用，二是确保安全。

## 1. 创建hosts文件

`cd /etc/ansible/playbooks`

hosts内容如下：
>$ cat hosts

```bash
[test]
192.168.10.101
192.168.10.102

[test:vars]
ansible_ssh_user=ubuntu
ansible_ssh_pass=ubuntu
ansible_sudo_pass=ubuntu
```
这里的账号密码sudo密码，在`[test:vars]`下定义并应用到`test`组的所有机器。

## 2. 创建create_user.yml

```bash
$ cat create_user.yml 
# create_user playbook
- hosts: test
  become: True
  user: ubuntu

  roles:
    - create_user
```

这里`hosts: test`中的`test`对应机器为`hosts`文件中的`test`组机器。

## 3. 创建roles对应的main.yml

创建目录，`mkdir -p roles/create_user/tasks`，注意这里的`roles/create_user`与create_user.yml的最后2行配置要一致。

main.yml内容如下：

>$  cat roles/create_user/tasks/main.yml

<!-- {% raw %} -->
```bash
# Generate random password for new_user_name and the new_user_name
# is required to change his/her password on first logon. 

- name: Generate password for new user
  #shell: echo -n "1qaz2wsx" # 如需设置每台机器的账号密码一样，可启用此行
                             # 注释下一行"shell: command openssl rand -base64 6"
  shell: command openssl rand -base64 6
  register: user_password

- name: Generate encrypted password for new user
  shell: command openssl passwd -salt '2018model' -1 {{ user_password.stdout }}
  register: encrypted_user_password

- name: Create user account
  user: 
      name: "{{ new_user_name }}"
      password: "{{ encrypted_user_password.stdout }}"
#password: "{{ 'password' | password_hash('sha512') }}"
      shell: /bin/bash
      update_password: on_create
      state: present
      groups: adm,sudo
  when: new_user_name is defined
  register: user_created

- name: Force user to change password
  shell: chage -d 0 {{ new_user_name }}
  when: user_created.changed

- name: User created
  debug: msg="Password for {{ new_user_name }} is {{ user_password.stdout }}"
  when: user_created.changed
```
<!-- {% endraw %} -->

## 4. 运行playbook添加账号

添加账号并对每个账号创建随机密码，强制密码过期，首次登录需修改密码。
> $ ansible-playbook -i hosts create_user.yml --extra-vars "new_user_name=test2018" -f 10 --sudo 

```
# -i  指定主机文件
# -f 10 设置并发数为10
# --sudo 使用sudo，
不建议这样做，建议使用-b, --become run operations with become -b参数为提权
# -extra-vars 
-e EXTRA_VARS, --extra-vars=EXTRA_VARS set additional variables as key=value or YAML/JSON
# 额外的变量设置，以"键=值"形式填写，或yaml、json形式。
```

```bash
$ansible-playbook -i hosts create_user.yml --extra-vars "new_user_name=test2018" -f 10 \
--sudo 

PLAY ***************************************************************************

TASK [setup] *******************************************************************
ok: [192.168.10.102]
ok: [192.168.10.101]

TASK [create_user : Generate password for new user] ****************************
changed: [192.168.10.101]
changed: [192.168.10.102]

TASK [create_user : Generate encrypted password for new user] ******************
changed: [192.168.10.101]
changed: [192.168.10.102]

TASK [create_user : Create user account] ***************************************
changed: [192.168.10.101]
changed: [192.168.10.102]

TASK [create_user : Force user to change password] *****************************
changed: [192.168.10.101]
changed: [192.168.10.102]

TASK [create_user : User created] **********************************************
ok: [192.168.10.101] => {
    "msg": "Password for test2018 is zkmuXatj"
}
ok: [192.168.10.102] => {
    "msg": "Password for test2018 is qTUPuvoi"
}

PLAY RECAP *********************************************************************
192.168.10.101             : ok=6    changed=4    unreachable=0    failed=0   
192.168.10.102             : ok=6    changed=4    unreachable=0    failed=0 
```

创建的各个机器的账号密码如下：

| IP | 账号 | 密码 |
| :-----------: | :-----------: | :------------: |
| 192.168.10.101 | test2018 | zkmuXatj |
| 192.168.10.102 | test2018 | qTUPuvoi |

## 5. 登陆验证

通过ssh连接192.168.10.101，会看到密码过期的提示(`Your password has expired`)，输入当前密码`zkmuXatj`-->输入

两遍新密码重置，之后重新使用新密码连接即可。

```bash
$ssh test2018@192.168.10.101
test2018@192.168.10.101's password: 
You are required to change your password immediately (root enforced)
Welcome to Ubuntu 14.04.5 LTS (GNU/Linux 4.4.0-31-generic x86_64)

The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

WARNING: Your password has expired.
You must change your password now and login again!
Changing password for test2018.
(current) UNIX password: 
Enter new UNIX password: 
Retype new UNIX password: 
passwd: password updated successfully
Connection to 192.168.10.101 closed.
```



## 参考
[creating-a-new-user-and-password-with-ansible](https://stackoverflow.com/questions/19292899/creating-a-new-user-and-password-with-ansible)

## 其他

可将host中的账号密码等信息放到group_vars目录下的一个test.yml，之后使用ansible-vault对test.yml加密，防止账号密码泄露，确保安全。
详细步骤可参考：

[Ansible vault example](https://gist.github.com/xoyabc/4ab27d181808affa6450ee481e0ff9b2)



