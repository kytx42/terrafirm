# Terrafoundry

[Terraform Docs](https://www.terraform.io/docs/index.html)

I created this project because I was tired of reading post after post after post
of how other people used Terraform and trying to adhere to a bunch of scattered
best practices. So I decided to add to that confusion with my own input.

I think this goes without saying, but do note that there are things specific to
the environments I work in that this module adheres to. I am not saying this is
best practice for your environments; all I am putting forth here is a wrapper
and structure for you to look over and see/modify it to fit for your use if necessary.

### Terraform Wrapper
To enforce my best practices within Terraform, I have written a Terraform wrapper
that has a handful of checks in it to ensure you are working within the structure
that we have setup for this repository. Run the script from the root directory of
this project (terraform.sh) with the -h argument or none and it will output the
'HELP' message explaining how to use it. In short:
```
./terraform.sh (action) (environment) (module) (optional_extra_tf_args)
```
This script does the following things (among potential others):
* Implements various failsafes to make sure you are adhering to best practices:
  * Makes sure you are in the root directory of this project before executing.
  * Makes sure that you have a valid .aws directory/symlink created in the project root.
  * Makes sure that you are specifying a valid Terraform action before executing.
  * Makes sure that you are specifying a valid (internal) module before executing.
  * Makes sure that you are specifying a valid environment before executing.
* Pulls (external) modules with Terraform "get" then validates your code before running.
* Sets up remote state files on S3 separated by environment and (internal) module.
  * Locks these state files if you are running this script -- in turn checks for
a lock to make sure nobody else is Terraforming the same (internal) module you are

### AWS Credentials
To setup AWS credentials, create a symlink of your AWS config directory to the 
infrastructure/.aws as follows:
```
ln -s $HOME/.aws/ terrafoundry/.aws
```
The values for this credentials file, profile, and region are all set in the
common sections of the .tfvars files.

### Module Structure
#### See the module_templates folder under the modules folder for examples.

By default, for every resource you create, you should call a module for that 
resource. The folder structure of an "external" module will look like this:
> external_module
>> main.tf

>> variables.tf

>> outputs.tf

The folder structure of an "internal" module will look similar to this:
> internal_module
>> .terraform/* (This is a git ignored file, it contains module symlinks which 
get created with Terraform get)

>> internal_module.tf

>> variables.tf -> ../../../variables/common.tfvars (This file is a symlink to 
common.tfvars, more info below)

### Variable Structure
In order to group common variables together easily, every internal module by 
default symlinks the common.tfvars file found in the root "variables" directory. 
From the internal module directory, symlink the 'common.tfvars' file:
```
ln -s ../../../variables/common.tfvars variables.tf
```

When using the wrapper to call a Terraform command, it will by default pull in 
the secondary variables file of the environment you have specified. For instance: 
```
./terraform.sh plan dev microservice
```
Will pull in the 'variables/dev.tfvars' file to override (if set/necessary) any 
defaults set forth in 'common.tfvars'.

### Testing and Validation
#### This section needs love.
There is a pre-commit hook script in the root of this directory. To enable it locally,
run this command from the project root:
```
ln -s pre-commit.sh .git/hooks/pre-commit
```