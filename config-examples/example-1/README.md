# Example 1 Documentation

There are 2 directories in this example, both with a single file that represents some config that might be similar to what you currently have as a TFVars file in one of your pipelines. 

The command `cto ecs config build --path config-examples/example-1/user-1 --recursive` will emit the complete config for user-1. 

```yaml
{
  "ec2": {
    "user-1-instance-a": {
      "instance_type": "t2.medium",
      "user_data": null,
      "ami": "ami-123456789",
      "instance_profile": "instance-profile-1",
      "disk_size": 200,
      "vpc_name": "default",
      "tags": {
        "terraform": true
      }
    },
    "user-1-instance-b": {
      "instance_type": "t2.2xlarge",
      "user_data": null,
      "ami": "ami-123456789",
      "instance_profile": "instance-profile-1",
      "disk_size": 200,
      "vpc_name": "default",
      "tags": {
        "terraform": true
      }
    }
  },
  "s3": {
    "user-1-bucket-a": {
      "private": true,
      "encrypted": true
    },
    "user-1-bucket-b": {
      "private": false,
      "encrypted": true
    }
  }
}
```

`cto ecs config build --path config-examples/example-1/user-1 --recusrive --filter ec2`

```yaml
{
    "user-1-instance-a": {
      "instance_type": "t2.medium",
      "user_data": null,
      "ami": "ami-123456789",
      "instance_profile": "instance-profile-1",
      "disk_size": 200,
      "vpc_name": "default",
      "tags": {
        "terraform": true
      }
    },
    "user-1-instance-b": {
      "instance_type": "t2.2xlarge",
      "user_data": null,
      "ami": "ami-123456789",
      "instance_profile": "instance-profile-1",
      "disk_size": 200,
      "vpc_name": "default",
      "tags": {
        "terraform": true
      }
    }
}
```

The data above might look familiar if you use for_each in Terraform to iterate over a set of resources obtained from TFVars with Terraform making changes based on changes in TFVars.

Alternatively, your Terraform might operate on a single instance at a time, so use the following to obtain a single instance. 

`cto ecs config build --path config-examples/example-1/pipeline-1 --recursive --filter 'ec2."user-1-instance-a"'`

Let's see how you would replace your TFVars file in your repo with central ECS based config. Let's use Jenkins as an example.

`Jenkinsfile`
```groovy
pipeline {
    parameters {
        string(name: 'USER', defaultValue: 'user-1', description: 'Which user do you wish to run this pipeline for?')
        string(name: 'RESOURCE', defaultValue: 'ec2', description: 'Which resource type do you wish to run this pipeline for?')
    }
    stages {
        stage('Retrieve configuration variables from ECS') {
            steps {
                echo 'Retrieving configuration variables from ECS' 
                sh "ecs config build --path "/config-examples/example-1/${params.USER}" --recursive --filter "${params.RESOURCE}" --out user.tfvars.json"
            }
        }
        stage('Running Terraform for user/ resource combination') {
            steps {
                echo "Running Terraform for user ${params.USER} resource ${params.RESOURCE}"
                sh "terraform apply --var-file user.tfvars.json --auto-approve"
            }
        }
    }
}
```

You can see it's very easy to switch from siloed config where pipeline variables are stored alongside pipeline code to a place where config lives in a central location uner the control of a config framework. 

This is only the beginning of course, it's not at all DRY (don't repeat yourself). There is no merging taking place in the example above, and merging is what we need to go DRY. 

Example 2 is a very basic example of merging, and example 6 achieves what we have above, but it's totally DRY so if all users objects share common variables, they are only specified once. Let's look at example 2 to get a first glimpse at merging. 