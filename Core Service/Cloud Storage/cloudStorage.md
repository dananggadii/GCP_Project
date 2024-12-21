# Cloud Storage

### 1. Preparation

Create a Cloud Storage bucket

1. On the Navigation menu (Navigation menu icon), click Cloud Storage > Buckets.

2. Click Create.

3. Specify the following, and leave the remaining settings as their defaults:

| Property                                        | Value                                                                               |
| ----------------------------------------------- | ----------------------------------------------------------------------------------- |
| Name                                            | Enter a globally unique name                                                        |
| Location                                        | type Region                                                                         |
| Region                                          | REGION                                                                              |
| Enforce public access prevention on this bucket | unchecked                                                                           |
| Access control                                  | Fine-grained (object-level permission in addition to your bucket-level permissions) |

4. Make a note of the bucket name. It will be used later in this lab and referred to as [BUCKET_NAME_1].

5. Click Create.

Download a sample file using CURL and make two copies

1. In the Cloud Console, click Activate Cloud Shell (Cloud Shell).

2. If prompted, click Continue.

3. Store [BUCKET_NAME_1] in an environment variable:

```bash
export BUCKET_NAME_1=<enter bucket name 1 here>
```

4. Verify it with echo:

```bash
echo $BUCKET_NAME_1
```

5. Run the following command to download a sample file (this sample file is a publicly available Hadoop documentation HTML file):

```bash
curl \
https://hadoop.apache.org/docs/current/\
hadoop-project-dist/hadoop-common/\
ClusterSetup.html > setup.html
```

6. To make copies of the file, run the following commands:

```bash
cp setup.html setup2.html
cp setup.html setup3.html
```

### 2. Access control lists (ACLs)

Copy the file to the bucket and configure the access control list

1. Run the following command to copy the first file to the bucket:

```bash
gcloud storage cp setup.html gs://$BUCKET_NAME_1/
```

2. To get the default access list that's been assigned to setup.html, run the following command:

```bash
gsutil acl get gs://$BUCKET_NAME_1/setup.html  > acl.txt
cat acl.txt
```

3. To set the access list to private and verify the results, run the following commands:

```bash
gsutil acl set private gs://$BUCKET_NAME_1/setup.html
gsutil acl get gs://$BUCKET_NAME_1/setup.html  > acl2.txt
cat acl2.txt
```

4. To update the access list to make the file publicly readable, run the following commands:

```bash
gsutil acl ch -u AllUsers:R gs://$BUCKET_NAME_1/setup.html
gsutil acl get gs://$BUCKET_NAME_1/setup.html  > acl3.txt
cat acl3.txt
```

Examine the file in the Cloud Console

1. In the Cloud Console, on the Navigation menu (Navigation menu icon), click Cloud Storage > Buckets.

2. Click [BUCKET_NAME_1].

3. Verify that for file setup.html, Public access has a Public link available.

Delete the local file and copy back from Cloud Storage

1. Return to Cloud Shell. If necessary, click Activate Cloud Shell (Cloud Shell).

2. Run the following command to delete the setup file:

```bash
rm setup.html
```

3. To verify that the file has been deleted, run the following command:

```bash
ls
```

4. To copy the file from the bucket again, run the following command:

```bash
gcloud storage cp gs://$BUCKET_NAME_1/setup.html setup.html
```
