## Overview
This repository contains a pipeline for converting genotype data from a CSV file into a VCF (Variant Call Format) file. The pipeline includes data processing steps to ensure proper formatting, handling missing values, and sorting SNPs by chromosome and position.

## Pipeline Description

### Input
The input to this pipeline is a CSV file containing SNP data with the following required columns:
- **id**: Unique identifier for each SNP.
- **Name**: Name or alternate identifier for each SNP.
- **REF**: Reference allele.
- **ALT**: Alternate allele.
- **CHROM**: Chromosome identifier.
- **POS**: Position of the SNP on the chromosome.
- Sample genotype columns: Each column contains genotype information for a specific sample. Missing values are represented by `--`.

### Steps to Generate VCF Files

#### 1. Conversion to VCF Format
The SNP data in the CSV file is converted into VCF format. The pipeline performs the following operations:
- **Header generation**: Adds VCF metadata and column headers (`#CHROM`, `POS`, `ID`, `REF`, `ALT`, `QUAL`, `FILTER`, `INFO`, `FORMAT`, and sample names).
- **Genotype conversion**: Converts sample genotype data:
  - `REF` repeated twice (e.g., `AA` for reference allele `A`) is converted to `0/0`.
  - `ALT` repeated twice (e.g., `GG` for alternate allele `G`) is converted to `1/1`.
  - A heterozygous genotype (e.g., `AG` for reference `A` and alternate `G`) is converted to `0/1`.
  - Missing values (`--`) are converted to `./.`.

#### 2. Sorting SNP Data
The SNP data is sorted by:
1. Chromosome (`CHROM`).
2. Position (`POS`).

This ensures that the VCF file adheres to the required format and order.

### Output
The pipeline generates the following VCF files:
1. **Original Converted VCF**: The VCF file directly converted from the input CSV.
2. **Updated VCF with Proper Missing Value Handling**: A corrected VCF file where missing values are represented as `./.`.
3. **Sorted VCF**: The SNP data is sorted by chromosome and position.


### Steps

1. Run the Python script provided in this repository.
   ```bash
   python diploidgeno2vcf.py --input <input_csv_file> --output <output_vcf_file>
   ```
2. The script generates the VCF files in the specified output location.

### Example
```bash
python diploidgeno2vcf.py --input 7k_SNIP_AR_raw.csv --output 7k_SNIP_AR_corrected.vcf
```

## Included Scripts

### diploidgeno2vcf.py

```python
import pandas as pd
import argparse

def convert_to_vcf(input_csv, output_vcf):
    """
    Convert a CSV file containing diploid genotype data to a VCF file.
    
    Parameters:
        input_csv (str): Path to the input CSV file.
        output_vcf (str): Path to the output VCF file.
    """
    # Load the input CSV file
    data = pd.read_csv(input_csv)

    # Prepare VCF header
    header = [
        "##fileformat=VCFv4.2",
        "##source=diploidgeno2vcf",
        "##INFO=<ID=DP,Number=1,Type=Integer,Description=\"Read Depth\">",
        "##FORMAT=<ID=GT,Number=1,Type=String,Description=\"Genotype\">",
        "#CHROM\tPOS\tID\tREF\tALT\tQUAL\tFILTER\tINFO\tFORMAT\t" + "\t".join(data.columns[6:])
    ]

    # Prepare VCF data rows
    vcf_rows = []
    for _, row in data.iterrows():
        # Extract genotype data and convert to VCF format
        genotypes = row.iloc[6:].apply(
            lambda gt: "./." if gt == "--" else (
                "0/0" if gt == row["REF"] * 2 else
                "1/1" if gt == row["ALT"] * 2 else
                "0/1"
            )
        )
        vcf_row = [
            row["CHROM"],  # CHROM
            row["POS"],    # POS
            row["id"],     # ID
            row["REF"],    # REF
            row["ALT"],    # ALT
            ".",           # QUAL (placeholder)
            "PASS",        # FILTER
            ".",           # INFO (placeholder)
            "GT",          # FORMAT
        ] + genotypes.tolist()
        vcf_rows.append("\t".join(map(str, vcf_row)))

    # Combine header and rows
    vcf_content = "\n".join(header + vcf_rows)

    # Write to the output VCF file
    with open(output_vcf, 'w') as file:
        file.write(vcf_content)

if __name__ == "__main__":
    # Set up argument parser
    parser = argparse.ArgumentParser(description="Convert diploid genotype CSV to VCF format.")
    parser.add_argument("--input", required=True, help="Path to the input CSV file.")
    parser.add_argument("--output", required=True, help="Path to the output VCF file.")

    # Parse arguments
    args = parser.parse_args()

    # Run the conversion
    convert_to_vcf(args.input, args.output)
```

## Notes
- Ensure the input CSV file has the required columns.
- Customize the script if additional processing steps are needed.

