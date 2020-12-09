# DynaTMT-py
The **DynaTMT tool** can be used to analyze **m**ultiplexed **e**nhanced **pro**tein **d**ynamic mass spectrometry (**mePROD**) data. mePROD uses pulse SILAC combined with Tandem Mass Tag (TMT) labelling to profile newly synthesized proteins. Through a booster channel, that contains a fully heavy labelled digest, the identification rate of labelled peptides is greatly enhanced, compared to other pSILAC experiments. Through the multiplexing capacity of TMT reagents it is possible during the workflow to use the boost signal as a carrier that improves survey scan intensities, but does not interfere with quantification of the pulsed samples. This workflow makes labelling times of minutes (down to 15min in the original publication) possible.
    Additionally, mePROD utilizes a baseline channel, comprised of a non-SILAC labelled digest that serves as a proxy for isolation interference and greatly improves quantification dynamic range. Quantification values of a heavy labelled peptide in that baseline channel are derived from co-fragmented heavy peptides and will be subtracted from the other quantifications. 
    For more information on mePROD, please refer to the [original publication](https://doi.org/10.1016/j.molcel.2019.11.010) Klann et al. 2020. 
The package can also be used to analyse any pSILAC/TMT dataset. 

## Usage
### Loading data
DynaTMT by default uses ProteomeDiscoverer Peptide or PSM file outputs in tab-delimited text. Relevant column headers are automatically extracted from the input file and processed accordingly.
    **Important Note:** DynaTMT assumes heavy labelled modifications to be named according to ProteomeDiscoverer or the custom TMT/SILAC lysine modification, respectively. The custom TMT/Lysine modification is necessary, since search engines are not compatible with two modifications on the same residue at the same time. Thus the heavy lysine as used during SILAC collides with the TMT modification at the lysine. To overcome this problem it is necessary to create a new chemical modification combining the two modification masses. Please name these modification as follows:
    
    *Label:13C(6)15N(4) – Heavy Arginine (PD default modification, DynaTMT searches for Label string in modifications)
    *TMTK8 – (Modification at lysines, +237.177 average modification mass)
    *TMTproK8 -  (Modification at lysines, +312.255 average modification mass)
    
    Alternatively, it is possible to input a plain text file containing Protein Accession or Identifiers in the first column, Ion Injection times in the second column (optional) and Peptide/PSM Modifications in the third column. All following columns are assumed to be TMT intensities, no matter the column names. For plain text files naming of the columns is irrelevant, as long as no duplicate column names are used.
You can change the all default column and modification names in the source code of the package if needed.

'''
import pandas as pd

df = pd.read_csv("PATH",sep='\t',header=0)
'''

## Workflow

mePROD uses injection time adjustment [Klann et al. 2020](https://doi.org/10.1021/acs.analchem.0c01749) as a first step, but that is optional.

In the default workflow the Input class is initialised with the input data. This data is stored in the class and gets modfified during normalisation and adjustments.

### IT adjustment
'''
from DynaTMT.DynaTMT import PD_input,plain_text_input
processor = PD_input(df)
processor.IT_adjustment()
'''
### Normalisation
Normalisation is performed either by total intensity normalisation, median normalisation or TMM.
Example (total intensity):
'''
processor.total_intensity_normalisation()
'''

### Extraction of heavy peptides
Here a dataframe is returned by the function
'''
extracted_heavy = processor.extract_heavy()
'''

**If you use normal pSILAC TMT data without mePROD baseline channels, you can stop here and extract also the light data, by calling extract_light()**

### Baseline normalisation
Here the baseline is subtracted from all samples and protein quantification rollup is performed.

'''
output = processor.extract_heavy(extracted_heavy,threshold=5,i_baseline=0,method='sum')
'''

### Store output
'''
output.to_csv("PATH",sep='\t')



# API documentation
    class PD_input
Class containing functions to analyze the default peptide/PSM output from ProteomeDiscoverer. All column names are assumed
to be default PD output. If your column names do not match these assumed strings, you can modify them or use the plain_text_input
class, that uses column order instead of names.