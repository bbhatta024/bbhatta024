#!/bin/bash
# Usage: Run this script in the bioinformaticsProject directory on CRC

# Step 1: Create an output directory if it doesn't exist
mkdir -p output 
# Step 2: Concatenate McrA and HSP70 sequences into single files
for file in ref_sequences/m*.fasta; do
    cat "$file" >> output/mcrAall.txt
done

for file in ref_sequences/h*.fasta; do
    cat "$file" >> output/hspall.txt
done

# Align McrA sequences
~/Private/Biocomputing/tools/muscle -align output/mcrAall.txt -output output/alignedmcrA.txt

# Align HSP70 sequences
~/Private/Biocomputing/tools/muscle -align output/hspall.txt -output output/alignedhsp.txt
# Step 4: Build HMM profiles using hmmbuild
~/Private/Biocomputing/tools/hmmbuild output/mcrA.hmm output/alignedmcrA.txt
~/Private/Biocomputing/tools/hmmbuild output/hsp.hmm output/alignedhsp.txt

# Step 5: Search each proteome file for McrA and HSP70 genes using hmmsearch
cd proteomes
for file in *.fasta; do
    ~/Private/Biocomputing/tools/hmmsearch -E 0.1 --tblout ../output/"${file%.fasta}".mcrAout ../output/mcrA.hmm "$file"
    ~/Private/Biocomputing/tools/hmmsearch -E 0.1 --tblout ../output/"${file%.fasta}".hspout ../output/hsp.hmm "$file"
Done

[bbhatta@crcfe01 proteomes]$ echo "name,mcrA_match,hsp_match" > ../Babita_informatics.csv
[bbhatta@crcfe01 proteomes]$ for file in *.fasta;
> do
> name="${file%.fasta}"
> mcrA_match=$(grep -vc "#" ../output/"$name".mcrAout)
> hsp_match=$(grep -vc "#" ../output/"$name".hspout)
> echo "$name,$mcrA_match,$hsp_match" >> ../Babita_informatics.csv
> done

# Step 7: Sort the output CSV file to determine the ideal candidates
sort -t, -k2,2nr -k3,3nr ../Yu_Zhu_Sharma_final.csv | sed '1!b' > ../sorted_candidates.csv


