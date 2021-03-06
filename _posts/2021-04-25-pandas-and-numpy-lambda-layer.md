---
layout: post
title:  "How to make a Python Pandas and Numpy Lambda layer"
date:   2021-04-25 19:18:01 +1000
categories: aws lambda layers python
---

Lambda layers are great, I can keep my lambda package size small and share the dependacy accross differnt functions.  One of the most commonly used set on librarys when dealing witt Python is both pandas and numpy.  In this overview I will show you how to make a layer that you can use.

Consider the following code:

```
import pandas as pd
import boto3
import os
import io
from io import StringIO

def lambda_handler(event, context):

    s3 = boto3.client('s3')
    obj = s3.get_object(Bucket = 'sample-bucket', Key = 'sample-file.tsv')
    raw_df = pd.read_csv(io.BytesIO(obj['Body'].read()), sep='\t'
```

The lmabda run time comes packaged with boto3 but not pandas or numpy awill result int he following errors:

Without pandas
```
Response
{
  "errorMessage": "Unable to import module 'app': No module named 'pandas'",
  "errorType": "Runtime.ImportModuleError",
  "stackTrace": []
}
```
and

without numpy
```
Response
{
  "errorMessage": "Unable to import module 'app': Unable to import required dependencies:\nnumpy: No module named 'numpy'",
  "errorType": "Runtime.ImportModuleError",
  "stackTrace": []
}
```
and

without pytz
```
{
  "errorMessage": "Unable to import module 'app': Unable to import required dependencies:\npytz: No module named 'pytz'",
  "errorType": "Runtime.ImportModuleError",
  "stackTrace": []
}
```

OK so let's build that layer.  I will build this for Python 3.8 but it's the same for 3.7 just use the 3.7 versions.

We will need 3 packages, they can be found at these 3 urls.

https://pypi.org/project/numpy/#files

https://pypi.org/project/pytz/#files

https://pypi.org/project/pandas/#files

The process is the same for all 3, brows to the URL and find the python versiopn you want and look for 

```
numpy-1.20.2-cp38-cp38-manylinux1_x86_64.whl (13.7 MB)
```
We want the:
```
x86_64
```
version.

Do this for all 3 and download those files

Next create a folder ( it can be in ~/Downlaods ) called python, the folder must be called python.

Then unzip each .whl file ( they are a whl extention but there a zip file )

Move the folders that are unzipped for each into the python folder.  For numpy there were 3 folders for pandas and pytz there were only 2.

The python folder shoudl now have 7 folders:

![Zip](/assets/post/2021-04-25-pandas-and-numpy-lambda-layer/layer-1.png "Layers")

Right click and zip this up ( the resulting file should be python.zip )

This is the layer and is ready for uploading.

Then Navigate to "Lambda" -> "Layers" and click "Create Layer", uplaod the ip file ( python.zip ) and dont forget to select the Compatible runtime.

From your lambda function scroll down to "Layers" and click "Add a Layer", select "Custom Layer" and you will see the layer you just created.  Add that and now you can use Numpy and Pandas in your lambda function

![Layer](/assets/post/2021-04-25-pandas-and-numpy-lambda-layer/layer-2.png "Layers")