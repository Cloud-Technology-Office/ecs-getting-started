# Example 5 Documentation

The prior examples are very useful, but sometimes we need more, such as generating lists of objects with IDs. This is exactly what we had in non DRY form in example 1, but how do we do that in a DRY fashion since the instance IDs are specified in the users individual config. Without using more advanced strategy logic, it is not possible to merge common config.

Use the following command to emit merged config using the strategy in <a href="__strategies.yaml">`__strategies.yaml`</a>: 

`cto ecs config build --path config-examples/example-5 --strategy-name generate_ec2s` 

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
    }
  }
}
```

The common config has been merged to each of the IDs as specified in the teams directories. Let's have a look at the `__strategies.yaml` file with some comments added so you can see how this is accomplished.

```yaml
strategies:
  generate_ec2s:
    # Generates a list of EC2s from contents of directories, with common_data merged in
    operations:
      # First we loop over the teams directories, see variable $directory_data to see where this array is coming from
      - for_each:
          iterator: $directory_data
          # You can reference the iterator directly with $each_key and $each_value in the loop directly inside the iterator. 
          # If you need to access the iterators data in loops that are not the direct descendent of the iterator, alias gives the iterator a name so it can be referenced directly. 
          # There are 3 iterator examples in the for_each loop below so you can see all of the options for data access in a direct descendent,
          # we're using $each_value as we can, but note that named iterators would be used for loops that are not direct descendents.
          alias: directory_iterator
          actions:
            - for_each:
                # $each_value here is the objects that live in each teams directory, we'll extract just the ec2 object which is in fact, the list of IDs and we'll iterate over those IDs
                iterator: $each_value.ec2
                actions:
                  - inject:
                      # $common_data is purely the ec2 object as we used source_object_path: ec2 to extract it from common
                      - $common_data
                      # $each_value is each ec2 object from the team config. It has the ID as the top level key
                      - $each_value
            # Let's add a parent to preserve the original structure with ec2 at the top. This is not necessary since this strategy is designed to work with just the ec2 object, but it illustrates the capability
            - add_parent: ec2

    variable_definitions:
      common_data:
        path: ./common
        # Using data_with_key_from_object_field instead of data allows us to manipulate the data read from common, in this case we only want the ec2 object
        type: data_with_key_from_object_field
        source_object_path: ec2
        destination_object_path: ''
      directory_data:
        path: ./teams/{team}
        # data_with_key_from_path allows us to use $each_key (the team name) and $each_value (the objects in that directory). In this example, we're only using $each_value
        type: data_with_key_from_path

```


