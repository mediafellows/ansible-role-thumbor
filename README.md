[![Build Status](https://travis-ci.org/mediapeers/ansible-role-thumbor.svg?branch=master)](https://travis-ci.org/mediapeers/ansible-role-thumbor)

# Thumbor role
Ansible role that installs [Thumbor](https://github.com/thumbor/thumbor) and sets it up for production use. It uses supervisord to spawn mulitple Thumbor servper processes and puts
Nnginx infront of it to loadbalance between them and provide a robus webserver for access from outside.

Also this role expects to use an S3 bucket as result storage and an S3 namespace as allowed image source.

This role is only designed to setup Thumbor up as an image scaling service. No uploading or other processing will be enabled. Also unsafe URLs are disbaled,
meaning you can only use the service with knowing the secret signing key. See `thumbor_signing_key` variable.

## Requirements
This roles is build for Ubuntu server 14.04 but also might work on other Debian based distros.
Also you need an AWS S3 bucket setup and the instance this runs on should assume an IAM role (or user credentials in .aws/) to make the
[AWS plugin](https://github.com/thumbor-community/aws) work (which uses [Boto](https://boto3.readthedocs.org/en/latest/guide/quickstart.html#configuration) to connect to S3).

## Role Variables
This is the list of role variables with their default values:

* `thumbor_signing_key: ABC123` - Overwrite this to make your thumbor secure! Key that's used to sign requests to Thumbor
* `thumbor_aws_plugin_version: 2.0.10` - Version of the (tc_aws plugin)[https://github.com/thumbor-community/aws] for thumbor
* `thumbor_user: ubuntu` - User that runs thumbor (through supervisord)
* `thumbor_config_dir: /etc/thumbor` - Dir that holds the thumbor config files
* `thumbor_log_dir: /var/log/thumbor`
* `thumbor_bucket_prefix: 'my-namespace-'` - Prefix for allowed image source S3 buckets
* `thumbor_client_side_cache_duration: 30` - client side cache duration in days
* `thumbor_result_storage_bucket: 'my-namespace-thumbor-cache'` - The bucket name for the result storage
* `thumbor_result_storage_path: result_storage` - The path (bucket folder) where results are cached
* `thumbor_result_storage_expiration: 24` - Result storage cache expiration time in hours
* `s3_aws_region: us-east-1` - AWS Region for S3 bucket (the aws plugin). If your instance assumes an IAM role you can set this and avoid an boto/aws config file completely
* `supervisord_log_dir: /var/log/supervisor` - Log dir for the supervisord service
* `nginx_graylog_server: log.server` - Graylog server for Nginx logs
* `nginx_log_dir: /var/log/nginx` - Nginx log dir
* `nginx_status_page: /nginx_status` - Nginx status page you can use for monitoring

There is a config for the Nginx role in `vars/main.yml`. It's set to work with thumbor supervisor setup. But you can throw out stuff you don't
need (such as graylog logging) if you want. Make sure you keep upstream servers in sync with the ones supervisor starts (thumbor/tornado servers).

## Dependencies
Depends on the Mediapeers Nginx role, which you can find here: https://github.com/mediapeers/ansible-role-nginx

## Example Playbook
This is an example on how to integrate this role into your playbook:
```yaml
- hosts: servers
  vars:
    thumbor_signing_key: 123ABC123Supersecret
    thumbor_bucket_prefix: "my-company-"
    thumbor_result_storage_bucket: "my-result-storage-bucket"
    # and other vars you want to override
  roles:
    - mpx.thumbor
  tasks:
    # other tasks
```

## License
BSD, as is.

## Author Information
Stefan Horning <horning@mediapeers.com>
