# ML practicals

This is the repository for ML practicals.

### setup a conda env and make it available to ipython

if you want to create an environment from scratch, that can be used in notebooks: 
[Source](https://medium.com/@nrk25693/how-to-add-your-conda-environment-to-your-jupyter-notebook-in-just-4-steps-abeab8b8d084)
```
conda create --name ml-labs
conda activate ml-labs
conda install -c anaconda ipykernel
python -m ipykernel install --user --name=ml-labs
```

in pycharm then:


- python 9
-  jupyter numpy scipy
 seaborn matplotlib pandas "pymc>=5"