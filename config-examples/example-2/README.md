# Example 2 Documentation

This example is structured in a way that allows basic merging. Using the `--recursive` flag with a nested directory structure will merge objects of the same name with objecvts higher in hierarchy taking presidence over objects lower down.

Use the following command to emit merged config: 

`cto ecs config build --path config-examples/example-2 --recursive` 

```yaml
{
  "ec2": {
    "instance_type": "small",
    "disk_size": 600
  }
}
```

As you can see, disk_size has been overriden by the config in `team-1` directory. 