# how-to: To create a read replica in AWS RDS for PostgreSQL
---
## Prerequisites:
- Source RDS instance must be running PostgreSQL and have automated backups enabled.
- The source DB must be in a supported version (PostgreSQL ≥ 9.3).
- If using multi-AZ, that's fine — read replicas work with both single-AZ and multi-AZ.
- Make sure network access (VPC/security groups) allows replication traffic.

# Option 1: Using AWS Console
- Go to the RDS Console.
- In the left menu, click Databases.
- Click the source DB instance (the one you want to replicate).
- In the Actions dropdown, select Create read replica.
- Configure the read replica:
- DB instance identifier (unique name).
- Instance class (e.g., db.t3.medium).
- Storage type (General Purpose SSD is usually fine).
- Enable Multi-AZ deployment if needed.
- Choose VPC, subnet group, and security groups.
- Set Public access depending on your use case.
- Click Create read replica.
AWS will now create a read-only replica of your DB.

# Option 2: Using AWS CLI

```
aws rds create-db-instance-read-replica \
    --db-instance-identifier my-read-replica \
    --source-db-instance-identifier my-source-db \
    --db-instance-class db.t3.medium \
    --availability-zone us-east-1a
```

You can also include options like --multi-az, --publicly-accessible, --tags, etc.

# Option 3: Using Terraform (if you're IaC-minded)

```
resource "aws_db_instance" "read_replica" {
  identifier              = "my-read-replica"
  replicate_source_db     = aws_db_instance.source.id
  instance_class          = "db.t3.medium"
  publicly_accessible     = false
  availability_zone       = "us-east-1a"
}

## Notes:
Read replicas are read-only — great for scaling read traffic.
You can promote a read replica to be a standalone writeable DB.
Replication is asynchronous, so there's eventual consistency.
You can also create cross-region read replicas, helpful for disaster recovery or latency reduction.
