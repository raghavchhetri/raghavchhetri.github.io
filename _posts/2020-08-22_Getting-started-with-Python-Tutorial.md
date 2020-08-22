
# Getting started with Python

<b>Raghav K. Chhetri<br/></b>
Keller Lab, HHMI Janelia<br/>
03/26/2020

    This tutorial is based on
    Python: v3.7
    OS: 64-bit Windows 10
    Make minor modifications wherever necessary if using other platforms

## Install Python 3.7 using Anaconda

1. Download [Anaconda installer](https://repo.anaconda.com/archive/) and proceed with installation

2. Once installation is complete, Python is ready to be launched. Two options:
    1. Graphical route > Anaconda Navigator  <br/>
    2. Command line > Anaconda Prompt -- we'll use this option in this tutorial

## Get familiar with Anaconda Prompt

From the Windows Start menu, search for and open “Anaconda Prompt”:

- Check your installation<br/>
    <code>(base) C:\Users\Raghav>where python</code>   


- Check Python version<br/>
    <code>(base) C:\Users\Raghav>python –-version</code>
    

- List all installed packages<br/>
    <code>(base) C:\Users\Raghav>conda list</code>
    

- Update to latest version<br/>
   <code>(base) C:\Users\Raghav>conda update conda </code>

## Start Python


Several options to do so:
- Command line execution using Python interpreter <br/> 
    <code>(base) C:\Users\Raghav>python</code>


- Interactive command line execution using IPython <br/> 
    <code>(base) C:\Users\Raghav>ipython</code>


- Spyder (matlab-like interface) <br/>
    <code>(base) C:\Users\Raghav>spyder</code>


- Jupyter notebook <br/>
    <code>(base) C:\Users\Raghav>jupyter notebook</code>

## Conda enviroments

- Create a new environment with a particular python version (<code>firstenv</code> in this example)<br/>
    <code>(base) C:\Users\Raghav>conda create –-name firstenv python=3.7 anaconda</code>


- Activate an environment <br/>
    <code>(base) C:\Users\Raghav>conda activate firstenv
    (firstenv)C:\Users\Raghav></code>


- Deactivate an active environment <br/>
    <code>(firstenv) C:\Users\Raghav>conda deactivate
    (base) C:\Users\Raghav></code>


- Clone from an existing environment <br/>
    <code>(base) C:\Users\Raghav>create --name secondenv –-clone firstenv</code>


- Delete an existing environment <br/>
    <code>conda env remove --name firstenv</code>


- List all environments <br/>
    <code>(base) C:\Users\Raghav>conda info --envs</code>


- [Conda Cheat Sheet](https://docs.conda.io/projects/conda/en/4.6.0/_downloads/52a95608c49671267e40c689e0bc00ca/conda-cheatsheet.pdf)

## pyklb

<code>pyklb</code> is a Python wrapper to read/write klb files. On 64-bit Windows running Python 3.7, install as follows: 

- Copy <code>Y:\Exchange\Raghav\tutorial_python\pyklb-master</code> to your local drive <br/>
    For example, to <code>C:\Users\<your_name></code>


- Launch Anaconda Prompt and activate your Python environment (<code>firstenv</code> in this example) <br/>
    <code>(base) C:\Users\Raghav>conda activate firstenv
    (firstenv) C:\Users\Raghav> </code>
    
    
- Navigate to the <code>\dist\</code> folder in your local <code>pyklb-master</code> <br/>
    <code>(firstenv) C:\Users\Raghav>cd pyklb-master\dist
    (firstenv) C:\Users\Raghav\pyklb-master\dist></code> 


- <code>pip</code> install the wheel present in this folder <br/>
    <code>(firstenv) C:\Users\Raghav\pyklb-master\dist>pip install pyklb-0.0.3.dev0-cp37-cp37m-win_amd64.whl</code> 


- Copy <code>Y:\Exchange\Raghav\tutorial_python\klb.dll</code> <br/>
    To your environment’s <code>\lib\site-packages\</code> folder <br/>
    For example, to <code>C:\Anaconda3\envs\firstenv\lib\site-packages</code>


If you are on a different OS or using a different Python version, build and install everything from scratch from [here](https://github.com/bhoeckendorf/pyklb)

### Open a .klb file in Python

- Let’s now open a .klb file in Python to confirm that things are working as expected. Below, I’m using <code>ipython</code> for this:

    <code>(base) C:\Users\Raghav>conda activate firstenv
    (firstenv) C:\Users\Raghav>ipython
    In [1]: import pyklb as klb
    In [2]: img = klb.readfull('y:/.../...klb’)
    In [3]: type(img)
    In [4]: img.shape
    In [5]: help(klb)</code>


- If this works, then you are all set with loading .klb files in your Python environment!
