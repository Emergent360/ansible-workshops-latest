# Workshop Exercise - Writing Your First Playbook

**Read this in other languages**:
<br>![uk](../../../images/uk.png) [English](README.md),  ![japan](../../../images/japan.png)[日本語](README.ja.md), ![brazil](../../../images/brazil.png) [Portugues do Brasil](README.pt-br.md), ![france](../../../images/fr.png) [Française](README.fr.md),![Español](../../../images/col.png) [Español](README.es.md).

## Table of Contents

* [Objective](#objective)
* [Guide](#guide)
  * [Step 1 - Playbook Basics](#step-1---playbook-basics)
  * [Step 2 - Creating a Directory Structure and File for your Playbook](#step-2---creating-a-directory-structure-and-file-for-your-playbook)
  * [Step 3 - Running the Playbook](#step-3---running-the-playbook)
  * [Step 4 - Extend your Playbook: Start &amp; Enable Apache](#step-4---extend-your-playbook-start--enable-apache)
  * [Step 5 - Extend your Playbook: Create an web.html](#step-5---extend-your-playbook-create-an-indexhtml)
  * [Step 6 - Practice: Apply to Multiple Host](#step-6---practice-apply-to-multiple-host)

## Objective

This exercise covers using Ansible to build two Apache web servers on Red Hat Enterprise Linux. This exercise covers the following Ansible fundamentals:

* Understanding Ansible Module parameters
* Understanding and using the following modules
  * [yum module](https://docs.ansible.com/ansible/latest/modules/yum_module.html)
  * [service module](https://docs.ansible.com/ansible/latest/modules/service_module.html)
  * [copy module](https://docs.ansible.com/ansible/latest/modules/copy_module.html)
* Understanding [Idempotence](https://en.wikipedia.org/wiki/Idempotence) and how Ansible Modules can be idempotent

## Guide

While Ansible ad hoc commands are useful for simple operations, they are not suited for complex configuration management or orchestration scenarios. For such use cases *playbooks* are the way to go.

Playbooks are files which describe the desired configurations or steps to implement on managed hosts. Playbooks can change lengthy, complex administrative tasks into easily repeatable routines with predictable and successful outcomes.

A playbook is where you can take some of those ad-hoc commands you just ran and put them into a repeatable set of *plays* and *tasks*.

A playbook can have multiple plays and a play can have one or multiple tasks. In a task a *module* is called, like the modules in the previous chapter. The goal of a *play* is to map a group of hosts.  The goal of a *task* is to implement modules against those hosts.

> **Tip**
>
> Here is a nice analogy: When Ansible modules are the tools in your workshop, the inventory is the materials and the Playbooks are the instructions.

### Step 1 - Playbook Basics

Playbooks are text files written in YAML format and therefore need:

* to start with three dashes (`---`)

* proper indentation using spaces and **not** tabs\!

There are some important concepts:

* **hosts**: the managed hosts to perform the tasks on

* **tasks**: the operations to be performed by invoking Ansible modules and passing them the necessary options.

* **become**: privilege escalation in Playbooks, same as using `-b` in the ad hoc command.

> **Warning**
>
> The ordering of the contents within a Playbook is important, because Ansible executes plays and tasks in the order they are presented.

A Playbook should be **idempotent**, so if a Playbook is run once to put the hosts in the correct state, it should be safe to run it a second time and it should make no further changes to the hosts.

> **Tip**
>
> Most Ansible modules are idempotent, so it is relatively easy to ensure this is true.

### Step 2 - Creating a Directory Structure and File for your Playbook

Enough theory, it’s time to create your first Ansible Playbook. In this lab you create a playbook to set up an Apache web server in three steps:

1. Install httpd package
2. Enable/start httpd service
3. Copy over an web.html file to each web host

This Playbook makes sure the package containing the Apache web server is installed on `node1`.

There is a [best practice](https://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html) on the preferred directory structures for playbooks.  We strongly encourage you to read and understand these practices as you develop your Ansible ninja skills.  That said, our playbook today is very basic and creating a complex structure will just confuse things.

Instead, we are going to create a very simple directory structure for our playbook, and add just a couple of files to it.

On your control host **ansible**, create a directory called `ansible-files` in your home directory and change directories into it:

```bash
[student<X>@ansible-1 ~]$ mkdir ansible-files
[student<X>@ansible-1 ~]$ cd ansible-files/
```

Add a file called `apache.yml` with the following content. As discussed in the previous exercises, use `vi`/`vim` or, if you are new to editors on the command line, check out the [editor intro](../0.0-support-docs/editor_intro.md) again.

```yaml
---
- name: Apache server installed
  hosts: node1
  become: yes
```

This shows one of Ansible’s strengths: The Playbook syntax is easy to read and understand. In this Playbook:

* A name is given for the play via `name:`.
* The host to run the playbook against is defined via `hosts:`.
* We enable user privilege escalation with `become:`.

> **Tip**
>
> You obviously need to use privilege escalation to install a package or run any other task that requires root permissions. This is done in the Playbook by `become: yes`.

Now that we've defined the play, let's add a task to get something done. We will add a task in which yum will ensure that the Apache package is installed in the latest version. Modify the file so that it looks like the following listing:

```yaml
---
- name: Apache server installed
  hosts: node1
  become: yes
  tasks:
  - name: latest Apache version installed
    yum:
      name: httpd
      state: latest
```

> **Tip**
>
> Since playbooks are written in YAML, alignment of the lines and keywords is crucial. Make sure to vertically align the *t* in `task` with the *b* in `become`. Once you are more familiar with Ansible, make sure to take some time and study a bit the [YAML Syntax](https://docs.ansible.com/ansible/latest/reference_appendices/YAMLSyntax.html).

In the added lines:

* We started the tasks part with the keyword `tasks:`.
* A task is named and the module for the task is referenced. Here it uses the `yum` module.
* Parameters for the module are added:
  * `name:` to identify the package name
  * `state:` to define the wanted state of the package

> **Tip**
>
> The module parameters are individual to each module. If in doubt, look them up again with `ansible-doc`.

Save your playbook and exit your editor.

### Step 3 - Running the Playbook

With the introduction of Ansible Automation Platform 2, several new key components are being introduced as a part of the overall developer experience. Execution Environments have been introduced to provide predictable environments to be used during automation runtime. All collection dependencies are contained within the Execution Environment to ensure that automation created in development environments runs the same as in production environments.

What do you find within an Execution Environment?

* RHEL UBI 8
* Ansible 2.9 or Ansible Core 2.11
* Python 3.8
* Any content Collections
* Collection python or binary dependencies.

Why use Execution Environments?

They provide a standardized way to define, build and distribute the environments that the automation runs in. In a nutshell, Automation execution environments are container images that allow for easier administration of Ansible by the platform administrator.

Considering the shift towards containerized execution of automation, automation development workflow and tooling that existed before Ansible Automation Platform 2 have had to be reimagined. In short, `ansible-navigator` replaces `ansible-playbook` and other `ansible-*` command line utilities.

With this change, Ansible Playbooks are executed using the `ansible-navigator` command on the control node.

The prerequisites and best practices for using `ansible-navigator` have been done for you within this lab.

These include:
* Installing the `ansible-navigator` package
* Creating a default settings `/home/student<X>/.ansible-navigator.yml` for all your projects (optional)
* All Execution Environment (EE) logs are stored within `/home/student<X>/.ansible-navigator/logs/ansible-navigator.log`
* Playbook artifacts are saved under `/tmp/artifact.json`

For more information on the [Ansible Navigator settings](https://github.com/ansible/ansible-navigator/blob/main/docs/settings.rst)

> **Tip**
>
> The parameters for Ansible Navigator maybe modified for your specific environment. The current settings use a default `ansible-navigator.yml` for all projects, but a specific `ansible-navigator.yml` can be created for each project and is the recommended practice. 

To run your playbook, use the `ansible-navigator run <playbook>` command as follows:

```bash
[student<X>@ansible-1 ansible-files]$ ansible-navigator run apache.yml
```

> **Tip**
>
> The existing `ansible-navigator.yml` file provides the location of your inventory file. If this was not set within your `ansible-navigator.yml` file, the command to run the playbook would be: `ansible-navigator run apache.yml -i /home/student<X>/lab_inventory/hosts`

When running the playbook, you'll be displayed a text user interface (TUI) that displays the play name among other information about the playbook that is currently run.

```bash
  PLAY NAME                        OK  CHANGED    UNREACHABLE      FAILED    SKIPPED    IGNORED    IN PROGRESS     TASK COUNT          PROGRESS
0│Apache server installed           2        1              0           0          0          0              0              2          COMPLETE
```

If you notice, prior to the play name `Apache server installed`, you'll see a `0`. By pressing the `0` key on your keyboard, you will be provided a new window view displaying the different tasks that ran for the playbook completion. In this example, those tasks included the "Gathering Facts" and "latest Apache version installed". The "Gathering Facts" is a built-in task that runs automatically at the beginning of each play. It collects information about the managed nodes. Exercises later on will cover this in more detail. The "latest Apache version installed" was the task created within the `apache.yml` file that installed `httpd`.

The display should look something like this:

```bash
  RESULT      HOST	NUMBER      CHANGED       TASK                                                   TASK ACTION           DURATION
0│OK          node1          0        False       Gathering Facts                                        gather_facts                1s
1│OK          node1          1         True       latest Apache version installed                        yum                         4s
```

Taking a closer look, you'll notice that each task is associated with a number. Task 1, "latest Apache version installed", had a change and used the `yum` module. In this case, the change is the installation of Apache (`httpd` package) on the host `node1`. 

By pressing `0` or `1` on your keyboard, you can see further details of the task being run. If a more traditional output view is desired, type `:st` within the text user interface.

Once you've completed, reviewing your Ansible playbook, you can exit out of the TUI via the Esc key on your keyboard.

> **Tip**
>
> The Esc key only takes you back to the previous screen. Once at the main overview screen an additional Esc key will take you back to the terminal window.


Once the playbook has completed, connect to `node1` via SSH to make sure Apache has been installed:

```bash
[student<X>@ansible-1 ansible-files]$ ssh node1
Last login: Wed May 15 14:03:45 2019 from 44.55.66.77
Managed by Ansible
```

Use the command `rpm -qi httpd` to verify httpd is installed:

```bash
[student<X>@node1 ~]$ rpm -qi httpd
Name        : httpd
Version     : 2.4.37
[...]
```

Log out of `node1` with the command `exit` so that you are back on the control host, and verify the installed package with an Ansible ad hoc command\!

```bash
[student<X>@ansible-1 ansible-files]$ ansible node1 -m command -a 'rpm -qi httpd'
```
```bash
node1 | CHANGED | rc=0 >>
Name        : httpd
Version     : 2.4.37
Release     : 39.module+el8.4.0+9658+b87b2deb
Architecture: x86_64
Install Date: Tue 27 Jul 2021 08:27:15 PM UTC
Group       : System Environment/Daemons
Size        : 4486208
License     : ASL 2.0
Signature   : RSA/SHA256, Mon 01 Feb 2021 07:24:18 PM UTC, Key ID 199e2f91fd431d51
Source RPM  : httpd-2.4.37-39.module+el8.4.0+9658+b87b2deb.src.rpm
Build Date  : Wed 27 Jan 2021 12:23:38 PM UTC
Build Host  : x86-vm-07.build.eng.bos.redhat.com
Relocations : (not relocatable)
Packager    : Red Hat, Inc. <http://bugzilla.redhat.com/bugzilla>
Vendor      : Red Hat, Inc.
URL         : https://httpd.apache.org/
Summary     : Apache HTTP Server
Description :
The Apache HTTP Server is a powerful, efficient, and extensible
web server.
```

Run the Playbook a second time via the `ansible-navigator run apache.yml`, and compare the output. The output "CHANGED" now shows `0` instead of `1` and the color changed from yellow to green. This makes it easier to spot when changes have occured when running the Ansible playbook.

### Step 4 - Extend your Playbook: Start & Enable Apache

The next part of the Ansible Playbook makes sure the Apache application is enabled and started on `node1`.

On the control host, as your student user, edit the file `~/ansible-files/apache.yml` to add a second task using the `service` module. The Playbook should now look like this:

```yaml
---
- name: Apache server installed
  hosts: node1
  become: yes
  tasks:
  - name: latest Apache version installed
    yum:
      name: httpd
      state: latest
  - name: Apache enabled and running
    service:
      name: httpd
      enabled: true
      state: started
```

What exactly did we do?

* a second task named "Apache enabled and running" is created
* a module is specified (`service`)
* The module `service` takes the name of the service (`httpd`), if it should be permanently set (`enabled`), and its current state (`started`)


Thus with the second task we make sure the Apache server is indeed running on the target machine. Run your extended Playbook:

```bash
[student<X>@ansible-1 ansible-files]$ ansible-navigator run apache.yml
```

Notice in the output, we see the play had `1` "CHANGED" shown in yellow and if we press `0` to enter the play output, you can see that task 2, "Apache enabled and running", was the task that incorporated the latest change by the "CHANGED" value being set to True and highlighted in yellow. 



* Use an Ansible ad hoc command again to make sure Apache has been enabled and started on `node1`, e.g. with: `systemctl status httpd`.

* Run the Playbook a second time using `ansible-navigator` to get used to the change in the output.

### Step 5 - Extend your Playbook: Create an web.html

Check that the tasks were executed correctly and Apache is accepting connections: Make an HTTP request using Ansible’s `uri` module in an ad hoc command from the control node. Make sure to replace the **\<IP\>** with the IP for the `node1` from the inventory.

> **Warning**
>
> **Expect a lot of red lines and a 403 status\!**

```bash
[student<X>@ansible-1 ansible-files]$ ansible localhost -m uri -a "url=http://<IP>"
```

There are a lot of red lines and an error: As long as there is not at least an `web.html` file to be served by Apache, it will throw an ugly "HTTP Error 403: Forbidden" status and Ansible will report an error.

So why not use Ansible to deploy a simple `web.html` file? On the ansible control host, as the `student<X>` user, create the directory `files` to hold file resources in `~/ansible-files/`:

```bash
[student<X>@ansible-1 ansible-files]$ mkdir files
```

Then create the file `~/ansible-files/files/web.html` on the control node:

```html
<body>
<h1>Apache is running fine</h1>
</body>
```

In a previous example, you used Ansible’s `copy` module to write text supplied on the command line into a file. Now you’ll use the module in your playbook to copy a file.

On the control node as your student user edit the file `~/ansible-files/apache.yml` and add a new task utilizing the `copy` module. It should now look like this:

```yaml
---
- name: Apache server installed
  hosts: node1
  become: yes
  tasks:
  - name: latest Apache version installed
    yum:
      name: httpd
      state: latest
  - name: Apache enabled and running
    service:
      name: httpd
      enabled: true
      state: started
  - name: copy web.html
    copy:
      src: web.html
      dest: /var/www/html/index.html
```

What does this new copy task do? The new task uses the `copy` module and defines the source and destination options for the copy operation as parameters.

Run your extended Playbook:

```bash
[student<X>@ansible-1 ansible-files]$ ansible-navigator run apache.yml
```

* Have a good look at the output, notice the changes of "CHANGED" and the tasks associated with that change. 

* Run the ad hoc command  using the "uri" module from above again to test Apache. The command should now return a friendly green "status: 200" line, amongst other information.

### Step 6 - Practice: Apply to Multiple Host

While the above, shows the simplicity of applying changes to a particular host. What about if you want to set changes to many hosts? This is where you'll notice the real power of Ansible as it applies the same set of tasks reliably to many hosts. 

* So what about changing the apache.yml Playbook to run on `node1` **and** `node2` **and** `node3`?

As you might remember, the inventory lists all nodes as members of the group `web`:

```ini
[web]
node1 ansible_host=11.22.33.44
node2 ansible_host=22.33.44.55
node3 ansible_host=33.44.55.66
```

> **Tip**
>
> The IP addresses shown here are just examples, your nodes will have different IP addresses.

Change the playbook `hosts` parameter to point to `web` instead of `node1`:

```yaml
---
- name: Apache server installed
  hosts: web
  become: yes
  tasks:
  - name: latest Apache version installed
    yum:
      name: httpd
      state: latest
  - name: Apache enabled and running
    service:
      name: httpd
      enabled: true
      state: started
  - name: copy web.html
    copy:
      src: web.html
      dest: /var/www/html/index.html
```

Now run the playbook:

```bash
[student<X>@ansible-1 ansible-files]$ ansible-navigator run apache.yml
```

Verify if Apache is now running on all web servers (node1, node2, node3). Identify the IP addresses of the nodes in your inventory first, and afterwards use them each in the ad hoc command with the uri module as we already did with the `node1` above. All output should be green.

> **Tip**
>
> Alternatively, verify that Apache is running on all web servers use the command `ansible web -m uri -a "url=http://localhost/"`.

---
**Navigation**
<br>
[Previous Exercise](../1.2-adhoc) - [Next Exercise](../1.4-variables)

[Click here to return to the Ansible for Red Hat Enterprise Linux Workshop](../README.md#section-1---ansible-engine-exercises)
