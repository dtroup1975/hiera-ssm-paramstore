[![Build Status](https://travis-ci.org/luizvasconceloss/hiera-ssm-paramstore.svg?branch=master)](https://travis-ci.org/luizvasconceloss/hiera-ssm-paramstore)

## hiera_ssm_paramstore : AWS Systems Manager Parameter Store backend for Hiera 5

### Description

This is a backend function for Hiera 5 that allows to lookup keys (string and securestring) on AWS Systems Manager Parameter Store.  The intent is to provide a more friendly way to manage keys on AWS.

### Compatibility

* It's only compatible with Hiera 5, present on Puppet 4.9+

### Requirements

The `aws-sdk-ssm` gem must be installed and loadable from Puppet

#### Puppet install
```puppet
include 'hiera_ssm_paramstore'
```

#### CLI install
```
# /opt/puppetlabs/puppet/bin/gem install aws-sdk-ssm
# puppetserver gem install aws-sdk-ssm
```

The server needs access to describe and get keys on AWS. You can use an `instance profile` or configure authentication through `aws-cli`. The policy should look like:

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "Stmt1526212635512",
            "Effect": "Allow",
            "Action": [
                "ssm:PutParameter", # -> only if you want write
                "ssm:AddTagsToResource", # -> only if you want write tags
                "ssm:GetParametersByPath",
                "ssm:GetParameters"
            ],
            "Resource": [
                "arn:aws:ssm:us-east-1:*",
                "arn:aws:ssm:us-east-2:*",
                "arn:aws:ssm:*:*:parameter/*"
            ]
        }
    ]
}
```

### Installation

Install directly via puppet or copy/clone to your modules directory

```
# puppet module install luizvasconceloss-hiera_ssm_paramstore
```

### Configuration

See [The official Puppet documentation](https://docs.puppet.com/puppet/4.9/hiera_intro.html) for more details on configuring Hiera 5.

The following is an example of Hiera 5 hiera.yaml configuration:

```yaml
---

version: 5

hierarchy:
  - name: "AWS Parameter Store"
    lookup_key: hiera_ssm_paramstore
    uris:
      - /
      - /hiera/%{facts.os.family}/
    options:
      region: us-east-1
      get_all: false
```


#### Lookup options

`region: ` : Specify what region should be used to query the keys/values, if not present will try to use a region configured on the server.<br/>
`get_all` : Get all key under the path (uri) and cache it, should reduce the api call and avoid throttle. Default value: false<br/>
`recursive` : Get all keys using recursive mode. Default value: false<br/>

#### Limitation

AWS impose rate limit for API call, depending on the number of keys and nodes you can quickly reach those limits.

#### Upgrading from version 0.1.x

Requires to update the IAM policy to use the option get_all

### As Puppet Functions

This module now also supports reading and writing ssm parameters as puppet functions within pp files.

```puppet
$options = {
  'uri'     => '/',
  'region'  => 'us-east-1',
  'get_all' => false,
  'put'     => { 'description' => 'Added by hiera_ssm_paramstore_write' },
}
$ssm_w_value = hiera_ssm_paramstore_write('/my/param', 'value', $options)
$ssm_r_value = hiera_ssm_paramstore('/my/param', $options)
```

#### Put options

[All options listed here](https://docs.aws.amazon.com/sdk-for-ruby/v2/api/Aws/SSM/Client.html#put_parameter-instance_method)

### Author

* Luiz Vasconcelos - luizvasconceloss01@gmail.com
