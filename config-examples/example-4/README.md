# Example 4 Documentation

This example is builds on example-3, instead of using a fixed object of `ec2`, it processes a list of objects that you are interested in compiling, in this case `ec2` and `s3`

Use the following command to emit merged config using the strategy in <a href="__strategies.yaml">`__strategies.yaml`</a>: 

`cto ecs config build --path config-examples/example-4 --strategy-name merge_common_objects_to_teams_objects` 

```yaml
{
  "team-1": {
    "ec2": {
      "instance_type": "t2.medium",
      "user_data": null,
      "ami": "ami-1234567890",
      "instance_profile": "instance-profile-1",
      "disk_size": 4000,
      "vpc_name": "default",
      "tags": {
        "terraform": true
      }
    },
    "s3": {
      "private": true,
      "encrytped": true
    }
  },
  "team-2": {
    "ec2": {
      "instance_type": "t3.medium",
      "user_data": null,
      "ami": "ami-1234567890",
      "instance_profile": "instance-profile-1",
      "disk_size": 20000,
      "vpc_name": "default",
      "tags": {
        "terraform": true
      }
    },
    "s3": {
      "private": false,
      "encrytped": true
    }
  }
}
```

As you can see, we're now merging common and emitting multiple objects.

