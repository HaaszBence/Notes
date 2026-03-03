### Donwloading the .ova

> To download the `.ova` go to the moodle page and find the following listed under the docker section

![download_page.png](assets/images/download_page.png)

> Download the  `.ova` by clicking `Pnet_lab`

![download_link.png](assets/images/download_link.png)

> When you click it you will be redirected to the following [OneDrive page](https://szaleziakk-my.sharepoint.com/personal/tothl_dk_akkszalezi_hu/_layouts/15/onedrive.aspx?id=%2Fpersonal%2Ftothl%5Fdk%5Fakkszalezi%5Fhu%2FDocuments%2FPnet%5FLab&ga=1)

![onedrive.png](assets/images/onedrive.png)

> When you select the `.ova` you will be prompted the following. We continue by ignoring the warning and clicking the download button.

![download_warning.png](assets/images/download_warning.png)

### Setting up the .ova

> Now that we have our `.ova` file we can use `Virtual Box` to import it. To do so select `file` in the top left corner and click on `Import Appliance`

![import.png](assets/images/import.png)

> Select the source of the `.ova` and click on `Finish`

> After you imported the virtual machine go into settings and enable `Nested VT-x/AMD-V`

![enable_nestedvirt.png](assets/images/enable_nestedvirt.png)

> After this you can start up the machine

### Using PNETLab

> To reach the `PNETLab` open a browser and in the search bar type [http://localhost](http://localhost)
> You will see the following

![login_page.png](assets/images/login_page.png)

>Enter the credentials also for the console select `HTML Console` and hit login

> Select your lab and press `open`

![assets/images/lab_select.png](assets/images/lab_select.png)

> To add new nodes right click and select `node`

![new_node.png](assets/images/new_node.png)