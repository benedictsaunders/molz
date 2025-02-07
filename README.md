# molZ 🧪

Statistical analysis tool to help identify molecular fragments that promote, or detract from,
target properties.

Sepecifically, this tool calculates the "z-scores" of molecular substructures in a given
sub-population of a database to identify fragments that are over- or under-represented in this
sub-population relative to a reference population. These substructures can either be specified
by the user, or automatically generated using Morgan fingerprints.

## How to install

molZ relies heavily on [RDKit](https://www.rdkit.org), which I recommend installing via conda
forge:

```bash
$ conda install -c conda-forge rdkit
```

Use the following to install the other prequisites:

```bash
$ pip install tqdm numpy scipy pandas pandasql matplotlib tabulate
```

After that, molZ can be installed with `pip`:

```bash
$ pip install molz
```

## How to use

Using auto-generated fragments:

```python
from molz import ZScorer

# instantiate scorer class, optionally set length and radius of morgan fingerprint.
# In this case, data.csv is a .CSV file of two columns: SMILES and computed LogP.
scorer = ZScorer('data.csv', fp_rad=3, fp_bits=4096)

# We are going to compute zscores of fragments present in high logp molecules.
# Once the ZScorer is initialised, we must set the property ranges; the data 
# column and upper and lower bounds are selected:
scorer.set_ranges([('penalised_logp', (12, 25))])

# Now we can compute the zscores
scorer.score_fragments()

# We can plot a bar graph of zscores for the 15 highest and lowest scoring fragments.
# Also, we can draw a given fragment by refering to its Morgan fingerprint bit index.
scorer.plot(k=15, save_to='zscores_auto.png')
scorer.draw_fragment(3595)

```

Using user-defined fragments:

```python
from molz import ZScorer

# instantiate scorer class. In this case, data.csv is a .CSV file of two columns:
# SMILES and computed LogP.
scorer = ZScorer('data.csv')

# We are going to compute zscores of fragments present in high logp molecules.
scorer.set ranges(
    [
        ('penalised_logp', (12, 25))
    ]
)
scorer.score_fragments(
    fragment_smiles=['CCCC', 'OC', 'N(C)(C)']
)

# We can plot a bar graph of zscores for the 15 highest and lowest scoring fragments.
# Also, we can draw a given fragment by refering to its SMILES.
scorer.plot(k=15, save_to='zscores_user.png')
scorer.draw_fragment('CCCC')
```

## Example of organic photovoltaics

We will use the data from ["Design Principles and Top Non-Fullerene Acceptor Candidates for
Organic Photovoltaics"](https://doi.org/10.1016/j.joule.2017.10.006) by Lopez et. al. as an
example.

First, we need the data, which comes from the article supplementary info:

```
$ curl https://ars.els-cdn.com/content/image/1-s2.0-S2542435117301307-mmc2.csv > lopez_data.csv
```

Now, we will use `molz` to detect over- and under-represented molecular fragments in molecues
with a predicted HOMO energy of less than than -6.3 eV and LUMO energy greater than -6.6 eV. 

We will use a relatively large number of fingerprint bits, to minimize
[bit collisions](http://rdkit.blogspot.com/2014/02/colliding-bits.html).

```python
from molz import ZScorer

# we will use the 'HOMO_calc' data column.
scorer = ZScorer('lopez-data.csv', fp_bits=8192, fp_rad=3)
scorer.set_ranges(
    [
        ("HOMO_calc", (-99, -6.3)),
        ("LUMO_calc", (-6.6, 99)),
    ]
)
scorer.score_fragments()
scorer.plot(k=40, figsize=(12, 3), save_to="lopez-homo-lumo.png", top_only=True, log_y=True)
```

Which gives the following plot:

<img src="./assets/lopez-homo-lumo.png"/>

We can the view each of the fragments:

```
scorer.draw_fragment(5607)
```

<img src="./assets/frag_5607.PNG"/>
