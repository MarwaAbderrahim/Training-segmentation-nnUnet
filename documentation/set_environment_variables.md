# How to set environment variables

nnU-Net requires some environment variables so that it always knows where the raw data, preprocessed data and trained 
models are. Depending on the operating system, these environment variables need to be set in different ways.

Variables can either be set permanently (recommended!) or you can decide to set them everytime you call nnU-Net. 

# Linux & MacOS

## Permanent
Locate the `.bashrc` file in your home folder and add the following lines to the bottom:

```bash
export nnUNet_raw="/media/test/nnUNet_raw"
export nnUNet_preprocessed="/media/test/nnUNet_preprocessed"
export nnUNet_results="/media/test/nnUNet_results"
```

(of course you need to adapt the paths to the actual folders you intend to use).
If you are using a different shell, such as zsh, you will need to find the correct script for it. For zsh this is `.zshrc`.

## Temporary
Just execute the following lines whenever you run nnU-Net:
```bash
export nnUNet_raw="/media/test/nnUNet_raw"
export nnUNet_preprocessed="/media/test/nnUNet_preprocessed"
export nnUNet_results="/media/test/nnUNet_results"
```
(of course you need to adapt the paths to the actual folders you intend to use).

Important: These variables will be deleted if you close your terminal! They will also only apply to the current 
terminal window and DO NOT transfer to other terminals!

Alternatively you can also just prefix them to your nnU-Net commands:

`nnUNet_results="/media/test/nnUNet_results" nnUNet_preprocessed="/media/test/nnUNet_preprocessed" nnUNetv2_train[...]`

## Verify that environment parameters are set
You can always execute `echo ${nnUNet_raw}` etc to print the environment variables. This will return an empty string if 
they were not set.

# Windows


## Temporary
Just execute the following before you run nnU-Net:

(powershell)
```powershell
$Env:nnUNet_raw = "/media/test/nnUNet_raw"
$Env:nnUNet_preprocessed = "/media/test/nnUNet_preprocessed"
$Env:nnUNet_results = "/media/test/nnUNet_results"
```

(command prompt)
```commandline
set nnUNet_raw=/media/test/nnUNet_raw
set nnUNet_preprocessed=/media/test/nnUNet_preprocessed
set nnUNet_results=/media/test/nnUNet_results
```

(of course you need to adapt the paths to the actual folders you intend to use).

Important: These variables will be deleted if you close your session! They will also only apply to the current 
window and DO NOT transfer to other sessions!

## Verify that environment parameters are set
Printing in Windows works differently depending on the environment you are in:

powershell: `echo $Env:[variable_name]`

command prompt: `echo %[variable_name]%`
