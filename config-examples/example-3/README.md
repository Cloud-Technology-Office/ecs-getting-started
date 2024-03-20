# Example 3 Documentation

This example is structured in a way that allows more advanced merging using a simple build strategy. There is no need to use the `--recursive` flag as the strategy takes care of all merge logic. 

This is where you start to see how real configs are used. Common config is typically stored above areas that override it with more specific config.

Use the following command to emit merged config using the strategy in <a href="__strategies.yaml">`__strategies.yaml`</a>: 

`cto ecs config build --path config-examples/example-3 --strategy-name merge_common_ec2_to_teams_ec2` 

```yaml
{
  "team-1": {
    "ec2": {
      "instance_type": "t2.medium",
      "user_data": null,
      "ami": "ami-123456789",
      "instance_profile": "instance-profile-1",
      "disk_size": 400,
      "vpc_name": "default",
      "tags": {
        "terraform": true
      }
    }
  },
  "team-2": {
    "ec2": {
      "instance_type": "t2.medium",
      "user_data": null,
      "ami": "ami-123456789",
      "instance_profile": "instance-profile-1",
      "disk_size": 200,
      "vpc_name": "vpc-team-2",
      "tags": {
        "terraform": true
      }
    }
  }
}
```

As you can see, the bulk of the config is in the common directory with just `disk_size` and `vpc_name` overriden per team. 

