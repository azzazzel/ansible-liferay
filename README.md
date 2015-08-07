Role Name
=========

An Ansible role to install and configure Liferay portal. 

Requirements & Limitations
------------

This role was tested only on Ansible 1.9.2 but may also work with older versions!
 
This is a very basic role and while it is capable of provisioning fully functional Liferay cluster, 
it has some significant limitations (comparing to what one can do configuring Liferay manually)!

 - Tested only on Linux and most likely will not work on windows (some tasks make use of linux command line utilities)
 - This role does not install Java. It simply assumes it's already installed. 
 - Currently it uses predefined `liferay-ext.properties` files and only allows to change particular values in it. 
 - Liferay clustering configuration is currently very limited:
   - it uses multicast for discovery 
   - it uses RMI threads for cache replication (Ehcache Cluster EE plugin not supported)
   - it uses replicated Lucene indexes for indexing/search (Solr is not supported)
   - document library is fixed to use `AdvancedFileSystemStore` and only file storage path is configurable
   - `cluster link` feature is always enabled (even if only a single machine is provisioned) 

Role Variables
--------------

```yaml
#
# Liferay archive to be installed. If the file in `local` exists it will be used. 
# If it doesn't exists and `url` is provided it will be downloaded
# It's user's responsibility to make sure the right file is downloaded 
# The role assumes the file is "liferay bundle" packaged as ZIP.
#
liferay_archive: {
  local: files/liferay-portal-tomcat-6.2-ce-ga4.zip,
  url: "http://sourceforge.net/projects/lportal/files/Liferay%20Portal/6.2.3%20GA4/liferay-portal-tomcat-6.2-ce-ga4-20150416163831865.zip" 
}

#
# The folder on the server whe the Liferay bundle will be unpacked 
#
liferay_unpack_folder: /usr/local

#
# Liferay's home folder. This is should not be changed unless you really know what you are doing! 
#
liferay_home: "{{ liferay_unpack_folder }}/{{ liferay_folder_from_zip.stdout }}"

#
# The operating system user and group that will be used to run Liferay 
#
liferay_os_user: liferay
liferay_os_group: liferay

#
# Liferay's default database 
#
liferay_default_database: {
  driver: "com.mysql.jdbc.Driver",
  url: "jdbc:mysql://localhost/lportal?useUnicode=true&characterEncoding=UTF-8&useFastDateParsing=false",
  user: "liferay",
  pass: liferay
}

#
# Additional databases to be added to Liferay's configuration 
#
liferay_additional_databases: [] 


#
# A folder where files from Liferay's document library will be stored.
# Change this to a shared storage in clustered environment 
#
liferay_dl_folder: "{{ liferay_home }}/data/document_library/"


#
# Network address to autodetect the default outgoing IP address 
# Change this to your database IP:PORT to make sure the right interface is used.
#
liferay_cluster_autodetect: google.com:80
```

Dependencies
------------

None

Example Playbook
----------------

The simplest form

    - hosts: servers
      roles:
         - role: milendyankov.liferay

will 
 
 * download `Liferay 6.2 CE GA4` bundled with Tomcat and save it `files/liferay-portal-tomcat-6.2-ce-ga4.zip`
 * copy and extract it in `/usr/local` on the remote server
 * configure `portal-ext.properties` to
   * disable startup wizard
   * use MySQL database called `lportal` on `localhost` 
   * enable cluster link feature
   
This is probably not what one would want, but it's a reasonable default. To configure working cluster 
with shared database and file system, something like this can be provided:

    - hosts: servers
      roles:
         - {
             role: milendyankov.liferay,
             liferay_archive: {
               local: <PATH_TO_STORE_FILE>,
               url: "<DOWNLOAD_URL>" 
             },
             liferay_default_database: {
               driver: "com.mysql.jdbc.Driver",
               url: "jdbc:mysql://<DATABASE_SERVER>/<DATABASE_NAME>?useUnicode=true&characterEncoding=UTF-8&useFastDateParsing=false",
               user: "<DATABASE_USER>",
               pass: "<DATABASE_PASSWORD>"
             },
             liferay_dl_folder: "/mnt/shared/liferay/document_library/",
             liferay_cluster_autodetect: <DATABASE_SERVER>:<DATABASE_PORT>
           }

If there are more databases Liferay needs to connect to, they can be added like this

    - hosts: servers
      roles:
         - {
             role: milendyankov.liferay,
             liferay_archive: {
               local: <PATH_TO_STORE_FILE>,
               url: "<DOWNLOAD_URL>" 
             },
             liferay_default_database: {
               driver: "com.mysql.jdbc.Driver",
               url: "jdbc:mysql://<DATABASE_SERVER>/<DATABASE_NAME>?useUnicode=true&characterEncoding=UTF-8&useFastDateParsing=false",
               user: "<DATABASE_USER>",
               pass: "<DATABASE_PASSWORD>"
             },
             liferay_dl_folder: "/mnt/shared/liferay/document_library/",
             liferay_cluster_autodetect: <DATABASE_SERVER>:<DATABASE_PORT>
             liferay_additional_databases:
              - {
                 id: custom1,
                 driver: "com.mysql.jdbc.Driver",
                 url: "jdbc:mysql://<DATABASE_SERVER>/<DATABASE_NAME>",
                 user: "<DATABASE_USER>",
                 pass: "<DATABASE_PASSWORD>"
	             }
              - {
                 id: custom2,
                 driver: "oracle.jdbc.OracleDriver",
                 url: "jdbc:oracle:thin:@//HOSTNAME:PORT/SERVICENAME",
                 user: "<DATABASE_USER>",
                 pass: "<DATABASE_PASSWORD>"
	             }
          }



License
-------

BSD

