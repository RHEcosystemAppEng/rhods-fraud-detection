### Prerequisite

- S3 bucket
- RHODS cluster

### Step 1: Download the openvino IR files
Download the files from your Notebook's file explorer to your local.

![download](gif/download-ir-files.gif)

Organise the downloaded files as shown below which is in accordance with guidelines of [openvino model repository](https://github.com/openvinotoolkit/model_server/blob/main/docs/models_repository.md#preparing-a-model-repository-ovms_docs_models_repository)

![structure](images/directory-structure.png)

### Step 2: Upload the files to your S3 bucket
Use your amazon console to upload files to your bucket.

![upload-to-s3](gif/upload-to-s3.gif)