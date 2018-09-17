
## Name

ec2-consistent-snapshot - Create EBS snapshots on EC2 with consistent filesystem

## Usage

    ec2-consistent-snapshot --description='' --tags="SomeKey=Some-Value;OtherKey=OtherValue"

## Description

This program creates creates EBS snapshots for all volumes attached to the current instance.
During the snapshots, all ext4 and xfs filesystems are frozen for consistency, typically just a few seconds.
As a safety measure, the root "/" partition is *not* frozen.

## Authentication

This program calls the `aws` binary to take the actual snapshot. Using an `IAM role` is recommended, requiring
no configuration files on the machine.

Here's an example Policy that can be used in an IAM role:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "1",
            "Effect": "Allow",
            "Action": [
                "ec2:CreateSnapshot",
                "ec2:CreateTags",
                "ec2:DescribeInstanceAttribute"
            ],
            "Resource": [
                "*"
            ]
        }
    ]
}
```

## Caveats

Be sure to test each filesystem you use it on before putting this into
production, in exactly the way you would run it in production (e.g., inside
cron if that's how you invoke it).

ec2-consistent-snapshot can hang if its output is directed at a filesystem that
is being frozen, leading to a dead machine. For that reason, we don't free the
root filesystem as an extra safety measure.

EBS snapshots are a critical part of protecting your valuable data. This
program or the environment in which it is run may contain defects that cause
snapshots to not be created. Please test and check to make sure that snapshots
are getting created for the volumes as you intend.

EBS snapshots cost money to create and to store in your AWS account. Be aware
of and monitor your expenses.

You are responsible for what happens in your EC2 account. This software is
intended, but not guaranteed, to help in that effort.

## Differences from ec2-consistent-snapshot (Perl version)

This tool is inspired by [Eric Hammond's
ec2-consistent-snapshot](https://github.com/alestic/ec2-consistent-snapshot), which is written in Perl.

This has the following advantages of the Perl-based ec2-consistent-snapshot:

 * We use a support AWS API client, while the Perl version uses an unsupported API client.
 * The Perl version sometimes experiences [authentication failures](https://github.com/mrallen1/net-amazon-ec2/issues/59) which remain
   unresolved. AWS Support is not interested to help address the issue with unsupported API client.
 * No options are provided here to flush a database first. MongoDB and MySQL and PostgreSQL all have
   some ability to recovery from unclean filesystem, so this may not be necessary.
 * Short, simple source code
 * Tag all volumes with a single command.
 * More flexible argument parsing-- we strictly require a "=" between keys and values, not a space.

However, the Perl version has it's own advantages:

  * Snapshot individual volumes
  * Options for verbose/dry-run/debug modes
  * Explicit support for MySQL and MongoDB

## Installation Dependencies

This tool is written in `bash` and depends on some other binaries that are are installed
by default on Ubuntu 14.04, 16.04, 18.04 and possibly other Linux variants.

 * bash > v4
 * sync
 * findmnt
 * curl
 * perl

 It also depends on the `aws` binary which is packaged for many Linux distributions and must be installed separately. On
 Ubuntu Linux:


    sudo apt install python3-pip && pip3 install --upgrade awscli

## License

Copyright 2018 Mark Stosberg <mark@rideamigos.com>


