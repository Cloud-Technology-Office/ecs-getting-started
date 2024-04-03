# Example 6 Documentation

Example 5 was a bit limited in that it is only working with a single object type. It is typically useful to compile an entire config structure and then use filtering to extract the relevant parts for the pipeline we're working with. Doing this saves writing a strategy for each object.

Use the following command to emit merged config using the strategy in <a href="__strategies.yaml">`__strategies.yaml`</a>: 

`cto ecs config build --path config-examples/example-6 --strategy-name generate_listed_objects` 

```yaml
{
  "team-1": {
    "ec2": {
      "server-1": {
        "instance_type": "t2.medium",
        "user_data": null,
        "ami": "ami-123456789",
        "instance_profile": "instance-profile-1",
        "disk_size": 400,
        "vpc_name": "default",
        "tags": {
          "terraform": true
        }
      },
      "server-2": {
        "instance_type": "t2.medium",
        "user_data": null,
        "ami": "ami-123456789",
        "instance_profile": "instance-profile-1",
        "disk_size": 800,
        "vpc_name": "default",
        "tags": {
          "terraform": true
        }
      }
    },
    "s3": {
      "bucket-123": {
        "private": true,
        "encrypted": true
      }
    }
  },
  "team-2": {
    "ec2": {
      "server-1": {
        "instance_type": "t2.medium",
        "user_data": null,
        "ami": "ami-123456789",
        "instance_profile": "instance-profile-1",
        "disk_size": 1400,
        "vpc_name": "default",
        "tags": {
          "terraform": true
        }
      },
      "server-2": {
        "instance_type": "t2.medium",
        "user_data": null,
        "ami": "ami-123456789",
        "instance_profile": "instance-profile-1",
        "disk_size": 1800,
        "vpc_name": "default",
        "tags": {
          "terraform": true
        }
      }
    },
    "s3": {
      "bucket-default": {
        "private": true,
        "encrypted": true
      },
      "bucket-private-false": {
        "private": false,
        "encrypted": true
      }
    }
  }
}
```

The common config has been merged to each of the IDs as specified in the teams directories. Let's have a look at the `__strategies.yaml` file with some comments added so you can see how this is accomplished.

```yaml
strategies:
  generate_listed_objects:
    # Generates a list of EC2 server and S3 bucket configs from contents of directories, with common_data merged in
    # Add objects to the list_iterator to work with more
    operations:
      - for_each:
          iterator: $directory_data
          alias: directory_iterator
          actions:
            - for_each: # this is a new loop that used list_iterator, the values here are literals and are referenced in the child loops
                list_iterator: [ 'ec2', 's3' ]
                alias: yaml_objects
                actions:
                  - for_each:
                      iterator: $directory_iterator#each_value.$yaml_objects#each_value
                      alias: instances
                      actions:
                        - inject:
                            - $common_data.$yaml_objects#each_value
                            - $directory_iterator#each_value.$yaml_objects#each_value.$each_key
                  - add_parent: $yaml_objects#each_value

    variable_definitions:
      common_data:
        path: ./common
        type: data
      directory_data:
        path: ./teams/{team}
        type: data_with_key_from_path
```

As mentioned, now we would use filtering to obtain the data we need from the overall config.

`cto ecs config build --path config-examples/example-6 --strategy-name generate_listed_objects --filter '"team-2".s3`

```yaml
{
  "bucket-default": {
    "private": true,
    "encrypted": true
  },
  "bucket-private-false": {
    "private": false,
    "encrypted": true
  }
}
```