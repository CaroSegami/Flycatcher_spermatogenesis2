# Documentation for improved annotation of the Ficedula albicollis Ensembl annotation version 103 (Genome assembly FicAlb 1.4)

## Extract protein sequences from Ensembl

Ficedula_albicollis.FicAlb_1.4.pep.all.fa.gz

## Download databses

1. Swissprot from Uniprot: <https://ftp.uniprot.org/pub/databases/uniprot/current_release/knowledgebase/complete/uniprot_sprot.fasta.gz>
2. TREMBL from Uniprot: <https://ftp.uniprot.org/pub/databases/uniprot/current_release/knowledgebase/complete/uniprot_trembl.fasta.gz> (2025-02-05 16:40-59G)

```bash
 #Concatenate
    zcat uniprot_sprot.fasta.gz uniprot_trembl.fasta.gz > uniprot_combined.fasta
#Then compress
    gzip uniprot_combined.fasta
```
# Run DIAMOND

I installed it in a conda environment.
First, make a database:

```bash
diamond makedb --in uniprot_combined.fasta.gz -d uniprot_combined
```

Then run diamond

```bash
diamond blastp -d uniprot_combined -q annotated_prot.fa -o dmnd_res_AnnProt_trembl+sprot.tsv --outfmt 6
```
Obtain gene names with full IDs (first col of results, here: dmnd_fullIDs_res_stand.txt)

```bash
zgrep ">" uniprot_combined.fasta.gz | grep -Ff dmnd_fullIDs_res_stand.txt > matched_headers_res_sub.fasta

echo "Get ID os and gn, first dataset"

awk 'BEGIN{OFS="\t"}
/^>/{ 
    full_id = substr($1, 2);  # Remove ">"
    gn = "NA";
    os = "NA";
    for (i = 1; i <= NF; i++) {
        if ($i ~ /^GN=/) {
            split($i, a, "=");
            gn = a[2];
        }
        if ($i ~ /^OS=/) {
            os = $i;
            # If species has spaces, grab the next word too
            if ((i+1) <= NF && $i !~ /.*;$/) {
                os = os " " $(i+1);
            }
            split(os, b, "=");
            os = b[2];
        }
    }
    print full_id, gn, os
}' matched_headers_res_sub.fasta > full_id_gn_os_res_sub.tsv

```