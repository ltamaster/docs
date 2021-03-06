% Upgrade Rundeck 2.11.X to Rundeck 3.X
% Luis Toledo
% June 1, 2018

### Rundeck Launcher

The "launcher jar" for Rundeck 2 is gone. However, the .war file now operates the same way. Just use the .war in the same way as the previous launcher jar, or deploy it as a webapp.

To upgrade to Rundeck 3 using launcher use the following steps:

* Stop Rundeck Service, eg: `.$RDECK_BASE/server/sbin/rundeckd stop`

* In case you have customs plugins on `libext` folder, backup them. For example, you can move the full `libext`:

```
mv $RDECK_BASE/libext $RDECK_BASE/libext.2-11
```

* Remove previous "source" folders: 

```
rm -rf server/exp/ server/lib/ server/sbin
```

* Copy the new war file to `$RDECK_BASE` and install it: 

```
java -jar rundeck-3.X.war --installonly
```

* Add the following attribute to `$RDECK_BASE/server/config/rundeck-config.properties`

```
rundeck.log4j.config.file=$RDECK_BASE/server/config/log4j.properties
```
*replace `$RDECK_BASE` with the real path.

* Copy the "custom" plugins back to `.$RDECK_BASE/libext` folder

* Start rundeck 3: `.$RDECK_BASE/server/sbin/rundeckd start`


### Rundeck DEB Package 

The upgrade process can be done using the `.deb` file or using the command `apt-get`:

* using deb package

```
sudo dpkg -i rundeck-3.0.X.deb
```


* from apt-get

```
sudo apt-get upgrade rundeck
```

**Configuration Files:**

When the upgrade command is run, the upgrade process will prompt about what to do with the new version of config files. In the case of `rundeck-config.properties`, a merge needs to be done between the old and new version, because the attribute `rundeck.log4j.config.file` is needed on Rundeck 3 (this attribute is part of the default `rundeck-config.properties` template):


```
Configuration file '/etc/rundeck/rundeck-config.properties'
 ==> Modified (by you or by a script) since installation.
 ==> Package distributor has shipped an updated version.
   What would you like to do about it ?  Your options are:
    Y or I  : install the package maintainer's version
    N or O  : keep your currently-installed version
      D     : show the differences between the versions
      Z     : start a shell to examine the situation
 The default action is to keep your current version.
*** rundeck-config.properties (Y/I/N/O/D/Z) [default=N] ?
```

For example, if you select "YES" the older file will be moved to `rundeck-config.properties.dpkg-old`, and `rundeck-config.properties` will be loaded with the default values (so a merge will be needed)

```
-rw-r----- 1 rundeck rundeck  986 May 11 19:13 realm.properties
-rw-r----- 1 rundeck rundeck  730 May 11 19:13 project.properties
-rw-r----- 1 rundeck rundeck 7529 May 11 19:13 log4j.properties
-rw-r----- 1 rundeck rundeck  136 May 11 19:13 jaas-loginmodule.conf
-rw-r----- 1 rundeck rundeck 1104 May 11 19:13 apitoken.aclpolicy
-rw-r----- 1 rundeck rundeck  738 May 11 19:13 admin.aclpolicy
-rw-r----- 1 rundeck rundeck 1438 Jun  1 19:28 framework.properties
-rw-r----- 1 rundeck rundeck  454 Jun  1 19:32 rundeck-config.properties
-rw-r----- 1 rundeck rundeck 3419 Jun  1 19:32 profile
-rw-r----- 1 rundeck rundeck  511 Jun  1 19:32 cli-log4j.properties
-rw-r----- 1 rundeck rundeck  673 Jun  4 14:22 rundeck-config.properties.dpkg-old
drwxr-x--- 1 rundeck rundeck 4096 Jun  4 14:25 ssl
```

A restart is necessary after the merge of `rundeck-config.properties`


### Rundeck RPM Package

The upgrade process can be done using the `.rmp` file or using the command `yum`:

* using rpm package

```
sh-4.2# rpm -U rundeck-3.x.rpm rundeck-config-3.X.rpm

warning: /etc/rundeck/framework.properties created as /etc/rundeck/framework.properties.rpmnew
warning: /etc/rundeck/rundeck-config.properties created as /etc/rundeck/rundeck-config.properties.rpmnew
rundeck.server.uuid = XXXXXXXXXXXXXXX

```

* Using yum

```
sh-4.2#  yum upgrade rundeck rundeck-config
```

In the case of the RPM upgrade, the old properties files are not modified, and the new files are saved with the extension `.rpmnew`. For the configuration file  `rundeck-config.properties` a merge is needed to include new default attribute (`rundeck.log4j.config.file`).

```
sh-4.2# ls -lrt
total 56
-rw-r----- 1 rundeck rundeck  455 Jun  1 19:34 rundeck-config.properties.rpmnew
-rw-r----- 1 rundeck rundeck  986 Jun  1 19:34 realm.properties
-rw-r----- 1 rundeck rundeck  729 Jun  1 19:34 project.properties
-rw-r----- 1 rundeck rundeck 3426 Jun  1 19:34 profile
-rw-r----- 1 rundeck rundeck 7538 Jun  1 19:34 log4j.properties
-rw-r----- 1 rundeck rundeck  136 Jun  1 19:34 jaas-loginmodule.conf
-rw-r----- 1 rundeck rundeck 1177 Jun  1 19:34 framework.properties.rpmnew
-rw-r----- 1 rundeck rundeck  511 Jun  1 19:34 cli-log4j.properties
-rw-r----- 1 rundeck rundeck 1104 Jun  1 19:34 apitoken.aclpolicy
-rw-r----- 1 rundeck rundeck  738 Jun  1 19:34 admin.aclpolicy
-rw-r----- 1 rundeck rundeck  673 Jun  4 16:02 rundeck-config.properties
-rw-r----- 1 rundeck rundeck 1505 Jun  4 16:02 framework.properties
drwxr-x--- 1 rundeck rundeck 4096 Jun  4 16:08 ssl
```

If the `profile` file was modified on 2.11.x, the new profile file (for 3.x) will be created with the name `profile.rpmnew`. If that is the case, merge the changes on the old file and move them to the new one, eg:

```
mv /etc/rundeck/profile.rpmnew /etc/rundeck/profile
```


A restart is necessary after the merge of `rundeck-config.properties` or/and `profile` file.




