<p align="center">
<b><a href="#free-gpu-ram-upgrade">Free GPU RAM Upgrade</a></b>
|
<b><a href="#prevent-disconnection">Prevent Disconnection</a></b>
|
<b><a href="#connect-to-gdrive">Connect to GDrive</a></b>
|
<b><a href="#import-custom-modules">Import Custom Modules</a></b>
|
<b><a href="#work-with-large-data">Work With Large Data</a></b>
|
<b><a href="#dump-data">Dump Data</a></b>
|
<b><a href="#display-image-inline">Display Image Inline</a></b>
|
<b><a href="#download-file">Download File</a></b>
</p>

# Google Colaboratory Management Routine 

### Free GPU RAM Upgrade 

Users is automatically alloted 12GB of RAM on Colab. However, this memory space is often difficult to work with for large dataset. To upgrade RAM resources to 25GB, we can execute the following code block: 

```python
a = []
while(1):
    a.append('1')
```

This infinite loop will cause the current session to crash, prompting Colab to ask if we want to upgrade to higher RAM space. Click "yes" and we're in!

Please use this sparingly, only when you really need more memory. Otherwise, I doubt Google will allow for this cheat code. Thank you! 

## Prevent Disconnection

Colab automatically disconnects your session if you don't interact with it after 30 minutes, causing all your data and variables to be lost. This is annoying when you're just training a model and don't want to sit there and babysit your kernel. 

To prevent this, we can run the following in the page console. To access this, right click on the page and choose Inspect (Command + Shift + C for Mac users). Then click on Console and paste the following code snippet:

```javascript
function ClickConnect(){
console.log("Working"); 
document.querySelector("colab-toolbar-button").click() 
}setInterval(ClickConnect,60000)
```

Then click Enter. Wait for a minute and see if "Working" is printed on the console before exiting. This code will simulate user interaction with the kernel every 60 seconds to prevent disconnection. 

## Save Model Checkpoints

It is good practice to save model weights as it trains to prevent having to start from scratch when Colab runtime runtime restarts during training. The following code helps to create a collection of checkpoint files that updates at the end of each epoch, saving best weights based on validation accuracy. 

More details for [TensorFlow](https://colab.research.google.com/github/tensorflow/docs/blob/master/site/en/tutorials/keras/save_and_load.ipynb#scrollTo=S_FA-ZvxuXQV). 

For TensorFlow:

- Save model checkpoints

```python

import os 
checkpoint_path = "training_1/cp.ckpt" 

checkpoint_dir = os.path.dirname(checkpoint_path)

 # Create checkpoint  callback 
cp_callback =ModelCheckpoint(checkpoint_path, 
     monitor='val_acc',save_best_only=True,save_weights_only=True,verbose=1)

network_fit = myModel.fit(x, y, batch_size=25, epochs=20,
                                  ,callbacks = [cp_callback] )
```

- Reload model with saved checkpoint weights:

```python

myModel.load_weights(checkpoint_path)
```

For Pytorch: 

- Save model checkpoints:

```python

model_save_name = 'classifier.pt'
path = f"/content/gdrive/My Drive/{model_save_name}" 
torch.save(model.state_dict(), path)
```

- Reload model with saved checkpoint weights:

```python

model_save_name = 'classifier.pt'
path = F"/content/gdrive/My Drive/{model_save_name}"
model.load_state_dict(torch.load(path))
```

## Connect to GDrive 

```python
from google.colab import drive
drive.mount('/content/gdrive')
```

## Import Custom Modules 

To import custom libraries and modules stored in Drive or Dropbox:

```python
import sys
sys.path.insert(0, module_path)

import your_module
```

## Work With Large Data

When working with large dataset with hundreds of thousands of files stored on GDrive, "[Errno 5] Input/output error" will often occur as GDrive operations time out due to the large number of files that it has to supply to Colab. To resolve this issue, the best way is to upload a zipped file with all the data that you need onto GDrive, connect Colab to GDrive and unloading that zipped file into the local environment of Colab. 

In this way, Colab can work with the dataset directly in its native environment, without having to reach back to Drive to access the file, which will cause major read time issue and IO error. 

```console
import os
!cp GDrive_path_to_zip_file Colab_path
os.chdir(Colab_path)
!unzip -qq zip_file
```

The above code will copy the zipped file in the GDrive path to the desired Colab local directory, and then unzip the file quietly so you can work with it directly in the Colab environment. 

## Dump Data

For long tail operations that needs large variable storage, Pickle seems to be the easiest option as it can store list, dictionary or any data object as binary files to be accessed for later use. Example:

To dump the clusters_rural variable for later use: 

```python
#Dump file for later integration with original dataframe
import os
import pickle

os.chdir('/content/drive/My Drive/Thesis/OSM_data')

with open('clusters_rural_pois', 'wb') as fp:
    pickle.dump(clusters_rural, fp)
```

To load back variables: 

```python
with open('clusters_rural_pois', 'rb') as fp:
  clusters_rural = pickle.load(fp)
```

However, this method is inefficient for image files variables as the binary format, for some reason, requires a huge amount of write-memory. I haven't figured out a better data storage method for this type. Please let me know if you have any idea. 

## Display Image Inline 

```python
from IPython.display import Image, display
Image('rnn-mnist.png')
```

## Download File

Sample operation:

```python

# plot model
plot_model(model, to_file='rnn-mnist.png', show_shapes=True)

# download model image file
from google.colab import files
files.download('rnn-mnist.png')
```

