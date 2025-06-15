# XLCAL
Hi here is XL-Calibur back looking camera calibration

# Instruction 
I attached my code here so you can try to run by yourself!

**Python Analysis:** 
The basic instruction about how to use XL-calibur data. 

**XL-CALI alignment data:** 
The code extracts and cleans alignment packets from XL-Calibur observation data, calculates how the center offsets and rotation angles evolve over time, and plots these curves to assess the instrument’s alignment stability.

**timespot classification:**
This code groups different time points during the observation based on image shift features and visualizes their distribution over time to help identify patterns in the instrument's motion.

The analysis results for different observation batches of Crab and Cygnus X-1 are all located in calibration result

The data you may need: the MJD time ranges corresponding to each observation batch are provided in XLCAL/calibration result/time spot list
