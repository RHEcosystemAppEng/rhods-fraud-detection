### Create Data connections, configure server and deploy the model
<ol type="a">
<li>Head over to your RHOAI dashboard to your Datascience project.</li>

click _Add data connection_</br>
![add-data-connection](images/add-data-connection.png)

Configure the data connection with respective values and click _Add data connection_
Note: the s3 endpoint might be different for your cluster. Usually, it is in the format https://s3.REGION.amazonaws.com</br>
![data-conn-config](gif/data-connection.png)

<li>Deploy the model to be served and **make sure** to make the route accessible externally.</li>

Click _Deploy_</br>
![configure-server](gif/deploy-model.gif)

Copy the inference link and head back to the [Notebook.ipynb](Notebook.ipynb)
</ol>