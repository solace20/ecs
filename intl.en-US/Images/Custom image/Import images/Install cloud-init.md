---
keyword: [cloud-init, Alibaba Cloud, image, custom, configuration]
---

# Install cloud-init

To ensure that the ECS instance running an image can initialize configurations, we recommend that you install cloud-init on the server when you create a custom Linux image. This topic describes how to install Alibaba Cloud cloud-init and the native cloud-init.

cloud-init is open source software used by cloud platforms to initialize Linux virtual machines. All major public cloud platforms such as Alibaba Cloud, Amazon Web Services \(AWS\), Microsoft Azure, and OpenStack support cloud-init. Alibaba Cloud cloud-init initializes the configurations of instances during their startup and executes user data scripts. These configurations include NTP, software sources, hostname, and SSH key pairs. For more information, visit [cloud-init Documentation](http://cloudinit.readthedocs.io/).

By default, cloud-init is installed for all Alibaba Cloud public images. To ensure that instances created from custom images can automatically initialize system configurations, we recommend that you install Alibaba Cloud cloud-init on your Linux server in the following scenarios:

-   Linux servers that will be migrated to the cloud but are not installed with cloud-init.

    **Note:** Proceed with caution when you install Alibaba Cloud cloud-init on servers that you do not plan to migrate to the cloud.

-   Linux servers that are installed with cloud-init of versions earlier than 0.7.9.
-   Alibaba Cloud ECS instances that are not installed with cloud-init.

## Description

Different cloud platforms may use different versions of cloud-init. Select the appropriate version and configure it with the appropriate data source \(datasource\). The latest version of Alibaba Cloud cloud-init is 19.1.2, and the data source is `Aliyun`.

**Note:** After cloud-init is installed, it automatically starts during the system startup. If the installed version of cloud-init is not compatible with the operating system of the server or the data source is not configured properly, cloud-init may not run normally and the instance may start slowly or even fail to start the next time you restart the instance. Therefore, you must select a later version of cloud-init and an appropriate data source such as `Aliyun`.

When you use cloud-init, take note of the following differences among different versions:

-   0.7.6a: the initial version of Alibaba Cloud cloud-init, which depends on python2.7 for the Python environment. Some early public images still use cloud-init 0.7.6a.

    **Note:** The Python community no longer provides technical support for python2.7. Therefore, we recommend that you use later versions of cloud-init to avoid potential risks associated with the dependency library.

-   0.7.9 and earlier: initial versions of the native cloud-init, which are not applicable to initialize ECS instances and must be upgraded.
-   18: cloud-init 18 and later automatically initialize network configurations. The code for network configuration is `BOOTPROTO=dhcp DEVICE=eth0 ONBOOT=yes STARTMODE=auto TYPE=Ethernet USERCTL=no`. If you want to customize network configurations after you install cloud-init, see the [\(Optional\) Customize network configuration](#section_v23_ilz_0cn) section in this topic.
-   19.1: Alibaba Cloud public images are upgrading to be installed with cloud-init 19.1, which depends on python3.6 for the Python environment.

## Check the cloud-init version

1.  Log on to the source server.

2.  Run the following command to check whether cloud-init is installed:

    ```
    which cloud-init
    ```

    If the output contains no path information, cloud-init is not installed. Install Alibaba Cloud cloud-init first.

3.  Run the following command to check the version of cloud-init:

    ```
    cloud-init --version
    ```

    If the returned version is earlier than 0.7.9, install Alibaba Cloud cloud-init.

4.  Back up data on the server.


## \(Recommended\) Install Alibaba Cloud cloud-init

Perform the following operations to download cloud-init 19.1.2 whose source code is `Aliyun`:

1.  Ensure that the python-pip dependency library is installed on the source server.

    Run the following commands to install the python3-pip dependency library for some Linux distributions:

    -   CentOS and Red Hat Enterprise Linux:

        ```
        yum -y install python3-pip
        ```

    -   Ubuntu and Debian:

        ```
        apt-get -y install python3-pip
        ```

    -   openSUSE and SUSE:

        ```
        zypper -n install python3-pip
        ```

2.  Run the following command to download the Alibaba Cloud cloud-init installation package:

    ```
    wget https://ecs-image-utils.oss-cn-hangzhou.aliyuncs.com/cloudinit/cloud-init-19.1.2.tgz
    ```

3.  Run the following command to decompress the cloud-init installation package to the current directory:

    ```
    tar -zxvf cloud-init-19.1.2.tgz
    ```

4.  Go to the cloud-init directory and install the dependency library.

    ```
    cd ./cloud-init-19.1.2
    pip3 install -r ./requirements.txt
    ```

5.  Go to the tools directory of the cloud-init file.

    ```
    cd ./tools
    ```

6.  Run the following command to execute the deploy.sh script to install cloud-init:

    ```
    bash ./deploy.sh <issue> <major_version>
    ```

    The following table describes the parameters and values in the deploy.sh script.

    |Parameter|Description|Example|
    |---------|-----------|-------|
    |issue|The type of the operating system. Valid values: centos, redhat, rhel, debian, ubuntu, opensuse, and sles. The parameter values are case-sensitive. sles stands for SUSE and SLES.|centos|
    |major\_version|The major version number of the operating system.|The major version number of CentOS 6.5 is 6.|

7.  Check whether cloud-init is installed.

    If `"description": "success"` is returned, Alibaba Cloud cloud-init is installed.

    ![Alibaba Cloud cloud-init installed](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/2963559951/p37025.png)


The following sample shell scripts used to install Alibaba Cloud cloud-init are provided for your reference. Adapt the scripts based on your actual operation system.

-   CentOS 6/7

    ```
    # Check whether the python3-pip dependency library is installed. If not, install it.
    if ! python3 -c 'import setuptools' >& /dev/null; then
      yum -y install python3-pip
    fi
    # Back up cloud-init of an earlier version.
    test -d /etc/cloud && mv /etc/cloud /etc/cloud-old
    # Download and decompress Alibaba Cloud cloud-init.
    wget https://ecs-image-utils.oss-cn-hangzhou.aliyuncs.com/cloudinit/cloud-init-19.1.2.tgz
    tar -zxvf ./cloud-init-19.1.2.tgz
    # Install cloud-init.
    issue_major=$( cat /etc/redhat-release | grep -Eo '[0-9]+\.?[ 0-9]+' | head -1 | awk -F'.' '{printf $1}')
    bash ./cloud-init-*/tools/deploy.sh centos "$issue_major"
    ```

-   Red Hat Enterprise Linux 6/7

    ```
    # Check whether the python3-pip dependency library is installed. If not, install it.
    if ! python3 -c 'import setuptools' >& /dev/null; then
      yum -y install python3-pip
    fi
    # Back up cloud-init of an earlier version.
    test -d /etc/cloud && mv /etc/cloud /etc/cloud-old
    # Download and decompress Alibaba Cloud cloud-init.
    wget https://ecs-image-utils.oss-cn-hangzhou.aliyuncs.com/cloudinit/cloud-init-19.1.2.tgz
    tar -zxvf ./cloud-init-19.1.2.tgz
    # Install cloud-init.
    issue_major=$( cat /etc/os-release | grep VERSION_ID | grep -Eo '[0-9]+\.?[ 0-9]+' | head -1 | awk -F'.' '{printf $1}')
    bash ./cloud-init-*/tools/deploy.sh rhel "$issue_major"
    ```

-   Ubuntu 14/16/18

    ```
    # Check whether the python3-pip dependency library is installed. If not, install it.
    if ! python3 -c 'import setuptools' >& /dev/null; then
      apt-get install python36 python3-pip -y
    fi
    # Back up cloud-init of an earlier version.
    test -d /etc/cloud && mv /etc/cloud /etc/cloud-old
    # Download and decompress Alibaba Cloud cloud-init.
    wget https://ecs-image-utils.oss-cn-hangzhou.aliyuncs.com/cloudinit/cloud-init-19.1.2.tgz
    tar -zxvf ./cloud-init-19.1.2.tgz
    # Install cloud-init.
    issue_major=$( cat /etc/os-release | grep VERSION_ID | grep -Eo '[0-9]+\.?[ 0-9]+' | head -1 | awk -F'.' '{printf $1}')
    bash ./cloud-init-*/tools/deploy.sh ubuntu "$issue_major"
    ```

-   Debian 8/9

    ```
    # Check whether the python3-pip dependency library is installed. If not, install it.
    if ! python3 -c 'import setuptools' >& /dev/null; then
      apt-get -y install python3-pip
    fi
    # Back up cloud-init of an earlier version.
    test -d /etc/cloud && mv /etc/cloud /etc/cloud-old
    # Download and decompress Alibaba Cloud cloud-init.
    wget https://ecs-image-utils.oss-cn-hangzhou.aliyuncs.com/cloudinit/cloud-init-19.1.2.tgz
    tar -zxvf ./cloud-init-19.1.2.tgz
    # Install cloud-init.
    issue_major=$( cat /etc/os-release | grep VERSION_ID | grep -Eo '[0-9]+\.?[ 0-9]+' | head -1 | awk -F'.' '{printf $1}')
    bash ./cloud-init-*/tools/deploy.sh debian "$issue_major"
    ```

-   SUSE 11/12

    ```
    # Check whether the python3-pip dependency library is installed. If not, install it.
    if ! python3 -c 'import setuptools'>& /dev/null; then
      zypper -n install python3-pip
    fi
    # Back up cloud-init of an earlier version.
    test -d /etc/cloud && mv /etc/cloud/etc/cloud-old
    # Download and decompress Alibaba Cloud cloud-init.
    wget https://ecs-image-utils.oss-cn-hangzhou.aliyuncs.com/cloudinit/cloud-init-19.1.2.tgz
    tar -zxvf ./cloud-init-19.1.2.tgz
    # Install cloud-init.
    issue_major=$( cat /etc/os-release | grep VERSION_ID | grep -Eo '[0-9]+\.?[ 0-9]+' | head -1 | awk -F'.' '{printf $1}')
    bash ./cloud-init-*/tools/deploy.sh sles "$issue_major"
    ```

-   OpenSUSE 13/42

    ```
    # Check whether the python3-pip dependency library is installed. If not, install it.
    if ! python3 -c 'import setuptools'>& /dev/null; then
      zypper -n install python3-pip
    fi
    # Back up cloud-init of an earlier version.
    test -d /etc/cloud && mv /etc/cloud/etc/cloud-old
    # Download and decompress Alibaba Cloud cloud-init.
    wget https://ecs-image-utils.oss-cn-hangzhou.aliyuncs.com/cloudinit/cloud-init-19.1.2.tgz
    tar -zxvf ./cloud-init-19.1.2.tgz
    # Install cloud-init.
    issue_major=$( cat /etc/os-release | grep VERSION_ID | grep -Eo '[0-9]+\.?[ 0-9]+' | head -1 | awk -F'.' '{printf $1}')
    bash ./cloud-init-*/tools/deploy.sh opensuse"$issue_major"
    ```


## \(Optional\) Install the native cloud-init

1.  Make sure that you have installed the Git, Python, and python-pip dependency libraries for the source server.

    Run the following commands to install Git, python3.6, and python3-pip dependency libraries for some Linux distributions:

    -   CentOS and Red Hat Enterprise Linux:

        ```
        yum -y install git python36 python3-pip
        ```

    -   Ubuntu and Debian:

        ```
        apt-get -y install git python36 python3-pip
        ```

    -   openSUSE and SUSE:

        ```
        zypper -n install git python36 python3-pip
        ```

2.  Run the following command to download the cloud-init source code package from Git:

    ```
    git clone https://git.launchpad.net/cloud-init
    ```

3.  Go to the cloud-init directory.

    ```
    cd ./cloud-init
    ```

4.  Run the following command to install all the dependency libraries:

    ```
    pip3 install -r ./requirements.txt
    ```

5.  Run the following command to install cloud-init:

    ```
    python3 setup.py install
    ```

6.  Modify the cloud.cfg configuration file.

    1.  Open the configuration file.

        ```
        vi /etc/cloud/cloud.cfg
        ```

        ![vi /etc/cloud/cloud.cfg](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/2963559951/p4621.png)

    2.  Modify the configuration before `cloud_init_modules:` as shown in the following section:

        ```
        # Example datasource config
        # The top level settings are used as module
        # and system configuration.
        # A set of users which may be applied and/or used by various modules
        # when a 'default' entry is found it will reference the 'default_user'
        # from the distro configuration specified below
        users:
           - default
        user:
            name: root
            lock_passwd: False
        # If this is set, 'root' will not be able to ssh in and they 
        # will get a message to login instead as the above $user
        disable_root: false
        # This will cause the set+update hostname module to not operate (if true)
        preserve_hostname: false
        syslog_fix_perms: root:root
        datasource_list: [ AliYun ]
        # Example datasource config
        datasource:
            AliYun:
                support_xen: false
                timeout: 5 # (defaults to 50 seconds)
                max_wait: 60 # (defaults to 120 seconds)
        #      metadata_urls: [ 'blah.com' ]
        # The modules that run in the 'init' stage
        cloud_init_modules:
        ```


## \(Optional\) Customize network configuration

1.  After cloud-init is installed, open the /etc/cloud/cloud.cfg file.

    ```
    vim /etc/cloud/cloud.cfg
    ```

2.  Add the disabled configuration before `Example datasource config`.

    ```
    network:
      config: disabled
    ```

    **Note:** After the configuration is added, you must manage the network configuration under the /etc/sysconfig/network-scripts/ directory.

    ![cloud-init-disable-config](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/3963559951/p73375.png)


## Troubleshooting

-   The libraries that are missing may vary depending on images. You can use pip to install the libraries and then install cloud-init again.
-   If the default software package manager \(such as yum\) and pip manager have been installed with different versions of dependency libraries, library version conflicts may occur and cause cloud-init to run abnormally. We recommend that you download the dependency libraries based on the error message.

|Error message|Description|Troubleshooting command|
|-------------|-----------|-----------------------|
|```
no setuptools module in python
```

|The python setuptools module is not installed.|The following examples use python3.6: -   CentOS and Red Hat: `yum -y install python3-pip`
-   Ubuntu and Debian: `apt-get -y install python3-pip`
-   openSUSE and SUSE: `zypper -n install python3-pip` |
|```
File "/root/cloud-init/cloudinit/log.py", line 19, in <module>
      import six
  ImportError: No module named six  )
```

|The six dependency library is not installed.|```
pip3 install six
``` |
|```
File "/root/cloud-init/cloudinit/url_helper.py", line 20, in <module>
      import oauthlib.oauth1 as oauth1
  ImportError: No module named oauthlib.oauth1  )
```

|The oauthlib dependency library is not installed.|```
pip3 install oauthlib
``` |
|No uninstalled dependency libraries specified|The error message is not mapped.|Run the following command to install all dependency libraries that are listed in the requirements.txt file of cloud-init: ```
pip3 install -r requirements.txt
``` |

-   For Linux servers that will be migrated to the cloud:

    You can migrate the servers by using the Cloud Migration tool or importing the custom images. For more information, see [Migrate a server to Alibaba Cloud by using the Cloud Migration tool]() or [Import custom images](/intl.en-US/Images/Custom image/Import images/Import custom images.md).

-   For Alibaba Cloud ECS instances that already run Linux custom images:

    You can restart the system to check the installation result. If the system automatically configures the hostname, software sources, and NTP, cloud-init is installed. For example, if the network configuration file shows the following content, cloud-init is installed:

    ```
    [root@iZbp1ios3psx4hoi******Z ~]# cat /etc/sysconfig/network-scripts/ifcfg-eth0
    # Created by cloud-init on instance boot automatically, do not edit.
    #
    BOOTPROTO=dhcp
    DEVICE=eth0
    ONBOOT=yes
    STARTMODE=auto
    TYPE=Ethernet
    USERCTL=no
    ```


**Related topics**  


[cloud-init official website - Alibaba Cloud \(AliYun\)](http://cloudinit.readthedocs.io/en/latest/topics/datasources/aliyun.html)

