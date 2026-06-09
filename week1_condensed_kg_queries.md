# Node Lookup: confirm APOE, ABCA1, TREM2 and Alzheimer’s exist and get their IDs/CUIs.

## Query 1: Find APOE nodes

### First query: 
```cypher
MATCH (n)
WHERE toLower(n.node_label) CONTAINS 'apoe'
RETURN labels(n) AS labels, n.node_label AS label, n.node_code AS code, n.CUI AS cui, n.SAB AS source
LIMIT 25;
```

Finding any node whose label contains apoe. it also finds junk like Capoeta because it contains “apoe”.

### Second query: 
```cypher
MATCH (n)
WHERE n.node_label = 'APOE gene'
RETURN labels(n) AS labels, n.node_label AS label, n.node_code AS code, n.CUI AS cui, n.SAB AS source;
```
Find only nodes exactly named APOE gene.

### Output: 
- 1 ["Gene"] "APOE gene" "613" "C1412481" "HGNC" 

- 2 ["Disease"] "APOE gene" "412642004" "C1412481" "SNOMEDCT_US"

## Query 2: Find ABCA1

### Query:
```cypher
MATCH (n)
WHERE n.node_label = 'ABCA1 gene'
RETURN labels(n) AS labels, n.node_label AS label, n.node_code AS code, n.CUI AS cui, n.SAB AS source;
```

### Output:
- 1 ["Gene"] "ABCA1 gene" "29" "C1412058" "HGNC"


## Query 3: Find TREM2

### Query: 

```cypher
MATCH (n)
WHERE n.node_label = 'TREM2 gene'
RETURN labels(n) AS labels, n.node_label AS label, n.node_code AS code, n.CUI AS cui, n.SAB AS source;
```

### Output:
- 1 ["Gene"] "TREM2 gene" "17761" "C1425067" "HGNC"


## Query 4: FInd Alzheimer

### First query: 
```cypher
MATCH (n)
WHERE toLower(n.node_label) CONTAINS "alzheimer"
RETURN labels(n) AS labels, n.node_label AS label, n.node_code AS code, n.CUI AS cui, n.SAB AS source
LIMIT 50;
```
### Output:
Big output

### Second query:
```cypher
MATCH (n:Disease)
WHERE toLower(n.node_label) CONTAINS 'alzheimer'
RETURN n.node_label AS label, n.node_code AS code, n.CUI AS cui, n.SAB AS source
ORDER BY label
LIMIT 100;
```
### Output: 
- 7 "Alzheimer's Disease" "26929004" "C0002395" "SNOMEDCT_US"

## Node Lookup Results:

| Entity | Found in CondensedKG? | Best node / note |
|---|---|---|
| APOE | Yes | APOE gene, HGNC, code 613, CUI C1412481 |
| ABCA1 | Yes | ABCA1 gene, HGNC, code 29, CUI C1412058 |
| TREM2 | Yes | TREM2 gene, HGNC, code 17761, CUI C1425067 |
| Alzheimer’s disease | Yes | Selected main node: Alzheimer's Disease, SNOMEDCT_US, code 26929004, CUI C0002395 |


# Relationship/path check
Test whether APOE connects directly or indirectly to Alzheimer’s in CondensedKG.

## APOE direct Alzheimer’s check:
```cypher
MATCH (g:Gene {node_label:'APOE gene'}), (d:Disease {node_label:"Alzheimer's Disease"})
MATCH (g)-[r]-(d)
RETURN g.node_label AS gene, type(r) AS relationship, d.node_label AS disease, r
LIMIT 25;
```
### Output:
- 1 "APOE gene" "gene_associated_with_disease" "Alzheimer's Disease" [:gene_associated_with_disease {evidence_class: " ", SAB: "NCI"}]
- 2 "APOE gene" "disease_has_associated_gene" "Alzheimer's Disease"
[:disease_has_associated_gene {evidence_class: " ", SAB: "NCI"}]

## ABCA1/TREM2 direct Alzheimer’s check

```cypher
MATCH (g:Gene {node_label:'ABCA1 gene'}), (d:Disease {node_label:"Alzheimer's Disease"})
MATCH (g)-[r]-(d)
RETURN g.node_label AS gene, type(r) AS relationship, d.node_label AS disease, r
LIMIT 25;
```
```cypher
MATCH (g:Gene {node_label:'TREM2 gene'}), (d:Disease {node_label:"Alzheimer's Disease"})
MATCH (g)-[r]-(d)
RETURN g.node_label AS gene, type(r) AS relationship, d.node_label AS disease, r
LIMIT 25;
```

### Output
Empty


## Result: 
CondensedKG contains a direct relationship between APOE gene and Alzheimer's Disease. CondensedKG supports a direct APOE–Alzheimer’s graph association. 

| Gene | Disease | Relationship | Source |
|---|---|---|---|
| APOE gene | Alzheimer's Disease | gene_associated_with_disease | NCI |
| APOE gene | Alzheimer's Disease | disease_has_associated_gene | NCI |

However, direct CondensedKG links to the selected general Alzheimer's Disease node were not found for ABCA1 or TREM2.

# Short path checks: 2-hop and 3-hop APOE/ABCA1/TREM2 to Alzheimer’s

## APOE 2-hop paths

### Query: 
```cypher
MATCH path = (g:Gene {node_label:'APOE gene'})-[*1..2]-(d:Disease {node_label:"Alzheimer's Disease"})
RETURN path
LIMIT 25;
```

### Output:
PNG output: images_neo4j_queries/APOE 2-hop_paths.png

- 1 "(:Gene {unit: , node_synonyms: APOE gene, lower_bound: , SAB: HGNC, CUI: C1412481, node_code: 613, node_label: APOE gene, upper_bound: , value: })-[:associated_with {evidence_class:  , SAB: HGNCHPO}]->(:EFO {unit: , node_synonyms: Amyloidosis, lower_bound: , SAB: EFO, node_label: Amyloidosis, CUI: C0002726, node_code: 1001875, upper_bound: , value: })<-[:RO {evidence_class:  , SAB: CSP}]-(:Disease {unit: , node_synonyms: Alzheimer's Disease, SAB: SNOMEDCT_US, lower_bound: , node_label: Alzheimer's Disease, node_code: 26929004, CUI: C0002395, upper_bound: , value: })"
- 2 "(:Gene {unit: , node_synonyms: APOE gene, lower_bound: , SAB: HGNC, CUI: C1412481, node_code: 613, node_label: APOE gene, upper_bound: , value: })-[:associated_with {evidence_class:  , SAB: HGNCHPO}]->(:EFO {unit: , node_synonyms: Amyloidosis, lower_bound: , SAB: EFO, node_label: Amyloidosis, CUI: C0002726, node_code: 1001875, upper_bound: , value: })<-[:RO {evidence_class:  , SAB: MTH}]-(:Disease {unit: , node_synonyms: Alzheimer's Disease, SAB: SNOMEDCT_US, lower_bound: , node_label: Alzheimer's Disease, node_code: 26929004, CUI: C0002395, upper_bound: , value: })"
- 3 "(:Gene {unit: , node_synonyms: APOE gene, lower_bound: , SAB: HGNC, CUI: C1412481, node_code: 613, node_label: APOE gene, upper_bound: , value: })-[:associated_with {evidence_class:  , SAB: HGNCHPO}]->(:EFO {unit: , node_synonyms: Amyloidosis, lower_bound: , SAB: EFO, node_label: Amyloidosis, CUI: C0002726, node_code: 1001875, upper_bound: , value: })-[:RO {evidence_class:  , SAB: MTH}]->(:Disease {unit: , node_synonyms: Alzheimer's Disease, SAB: SNOMEDCT_US, lower_bound: , node_label: Alzheimer's Disease, node_code: 26929004, CUI: C0002395, upper_bound: , value: })"
- 4 "(:Gene {unit: , node_synonyms: APOE gene, lower_bound: , SAB: HGNC, CUI: C1412481, node_code: 613, node_label: APOE gene, upper_bound: , value: })-[:associated_with {evidence_class:  , SAB: HGNCHPO}]->(:EFO {unit: , node_synonyms: Amyloidosis, lower_bound: , SAB: EFO, node_label: Amyloidosis, CUI: C0002726, node_code: 1001875, upper_bound: , value: })-[:RO {evidence_class:  , SAB: CSP}]->(:Disease {unit: , node_synonyms: Alzheimer's Disease, SAB: SNOMEDCT_US, lower_bound: , node_label: Alzheimer's Disease, node_code: 26929004, CUI: C0002395, upper_bound: , value: })"
- 5 "(:Gene {unit: , node_synonyms: APOE gene, lower_bound: , SAB: HGNC, CUI: C1412481, node_code: 613, node_label: APOE gene, upper_bound: , value: })-[:gene_associated_with_disease {evidence_class:  , SAB: NCI}]->(:Disease {unit: , node_synonyms: Alzheimer's Disease, SAB: SNOMEDCT_US, lower_bound: , node_label: Alzheimer's Disease, node_code: 26929004, CUI: C0002395, upper_bound: , value: })"
- 6 "(:Gene {unit: , node_synonyms: APOE gene, lower_bound: , SAB: HGNC, CUI: C1412481, node_code: 613, node_label: APOE gene, upper_bound: , value: })<-[:disease_has_associated_gene {evidence_class:  , SAB: NCI}]-(:Disease {unit: , node_synonyms: Alzheimer's Disease, SAB: SNOMEDCT_US, lower_bound: , node_label: Alzheimer's Disease, node_code: 26929004, CUI: C0002395, upper_bound: , value: })"
- 7 "(:Gene {unit: , node_synonyms: APOE gene, lower_bound: , SAB: HGNC, CUI: C1412481, node_code: 613, node_label: APOE gene, upper_bound: , value: })<-[:inverse_associated_with {evidence_class:  , SAB: HGNCHPO}]-(:EFO {unit: , node_synonyms: Amyloidosis, lower_bound: , SAB: EFO, node_label: Amyloidosis, CUI: C0002726, node_code: 1001875, upper_bound: , value: })<-[:RO {evidence_class:  , SAB: CSP}]-(:Disease {unit: , node_synonyms: Alzheimer's Disease, SAB: SNOMEDCT_US, lower_bound: , node_label: Alzheimer's Disease, node_code: 26929004, CUI: C0002395, upper_bound: , value: })"
- 8 "(:Gene {unit: , node_synonyms: APOE gene, lower_bound: , SAB: HGNC, CUI: C1412481, node_code: 613, node_label: APOE gene, upper_bound: , value: })<-[:inverse_associated_with {evidence_class:  , SAB: HGNCHPO}]-(:EFO {unit: , node_synonyms: Amyloidosis, lower_bound: , SAB: EFO, node_label: Amyloidosis, CUI: C0002726, node_code: 1001875, upper_bound: , value: })<-[:RO {evidence_class:  , SAB: MTH}]-(:Disease {unit: , node_synonyms: Alzheimer's Disease, SAB: SNOMEDCT_US, lower_bound: , node_label: Alzheimer's Disease, node_code: 26929004, CUI: C0002395, upper_bound: , value: })"
- 9 "(:Gene {unit: , node_synonyms: APOE gene, lower_bound: , SAB: HGNC, CUI: C1412481, node_code: 613, node_label: APOE gene, upper_bound: , value: })<-[:inverse_associated_with {evidence_class:  , SAB: HGNCHPO}]-(:EFO {unit: , node_synonyms: Amyloidosis, lower_bound: , SAB: EFO, node_label: Amyloidosis, CUI: C0002726, node_code: 1001875, upper_bound: , value: })-[:RO {evidence_class:  , SAB: MTH}]->(:Disease {unit: , node_synonyms: Alzheimer's Disease, SAB: SNOMEDCT_US, lower_bound: , node_label: Alzheimer's Disease, node_code: 26929004, CUI: C0002395, upper_bound: , value: })"
- 10 "(:Gene {unit: , node_synonyms: APOE gene, lower_bound: , SAB: HGNC, CUI: C1412481, node_code: 613, node_label: APOE gene, upper_bound: , value: })<-[:inverse_associated_with {evidence_class:  , SAB: HGNCHPO}]-(:EFO {unit: , node_synonyms: Amyloidosis, lower_bound: , SAB: EFO, node_label: Amyloidosis, CUI: C0002726, node_code: 1001875, upper_bound: , value: })-[:RO {evidence_class:  , SAB: CSP}]->(:Disease {unit: , node_synonyms: Alzheimer's Disease, SAB: SNOMEDCT_US, lower_bound: , node_label: Alzheimer's Disease, node_code: 26929004, CUI: C0002395, upper_bound: , value: })"

## ABCA1 2-hop paths

### Query: 
```cypher
MATCH path = (g:Gene {node_label:'ABCA1 gene'})-[*1..2]-(d:Disease {node_label:"Alzheimer's Disease"})
RETURN path
LIMIT 25;
```
### Output:
No output

## TREM2 2-hop paths

### Query:
```cypher
MATCH path = (g:Gene {node_label:'TREM2 gene'})-[*1..2]-(d:Disease {node_label:"Alzheimer's Disease"})
RETURN path
LIMIT 25;
```

### Output:
No output

## ABCA1 3-hop paths

### Query:
- PNG query: 
```cypher
MATCH path = (g:Gene {node_label:'ABCA1 gene'})-[*1..3]-(d:Disease {node_label:"Alzheimer's Disease"})
RETURN path
LIMIT 25;
```
- Summary output query:
```cypher
MATCH path = (g:Gene {node_label:'ABCA1 gene'})-[*1..3]-(d:Disease {node_label:"Alzheimer's Disease"})
WITH path, nodes(path) AS ns, relationships(path) AS rs
RETURN
  [n IN ns | n.node_label] AS nodes,
  [r IN rs | type(r)] AS relationships,
  [r IN rs | r.SAB] AS sources
LIMIT 25;
```

### Output:
PNG: Output is in images_neo4j_queries/ABCA1 3-hop_paths.png
Output: 
nodes	relationships	sources
[ABCA1 gene, Neuropathy, Nervous system structure, Alzheimer's Disease]	[associated_with, is_associated_anatomic_site_of, is_associated_anatomic_site_of]	[HGNCHPO, NCI, NCI]
[ABCA1 gene, Neuropathy, Nervous system structure, Alzheimer's Disease]	[associated_with, is_associated_anatomic_site_of, disease_has_associated_anatomic_site]	[HGNCHPO, NCI, NCI]
[ABCA1 gene, Neuropathy, GRN gene, Alzheimer's Disease]	[associated_with, associated_with, gene_associated_with_disease_or_phenotype]	[HGNCHPO, HGNCHPO, CLINVAR]
[ABCA1 gene, Neuropathy, GRN gene, Alzheimer's Disease]	[associated_with, associated_with, inverse_gene_associated_with_disease_or_phenotype]	[HGNCHPO, HGNCHPO, CLINVAR]
[ABCA1 gene, Neuropathy, MAPT gene, Alzheimer's Disease]	[associated_with, associated_with, gene_associated_with_disease_or_phenotype]	[HGNCHPO, HGNCHPO, CLINVAR]
[ABCA1 gene, Neuropathy, MAPT gene, Alzheimer's Disease]	[associated_with, associated_with, inverse_gene_associated_with_disease_or_phenotype]	[HGNCHPO, HGNCHPO, CLINVAR]
[ABCA1 gene, Neuropathy, PSEN1 gene, Alzheimer's Disease]	[associated_with, associated_with, gene_associated_with_disease]	[HGNCHPO, HGNCHPO, NCI]
[ABCA1 gene, Neuropathy, PSEN1 gene, Alzheimer's Disease]	[associated_with, associated_with, gene_associated_with_disease_or_phenotype]	[HGNCHPO, HGNCHPO, CLINVAR]
[ABCA1 gene, Neuropathy, PSEN1 gene, Alzheimer's Disease]	[associated_with, associated_with, inverse_gene_associated_with_disease_or_phenotype]	[HGNCHPO, HGNCHPO, CLINVAR]
[ABCA1 gene, Neuropathy, PSEN1 gene, Alzheimer's Disease]	[associated_with, associated_with, disease_has_associated_gene]	[HGNCHPO, HGNCHPO, NCI]
[ABCA1 gene, Neuropathy, VCP gene, Alzheimer's Disease]	[associated_with, associated_with, gene_associated_with_disease_or_phenotype]	[HGNCHPO, HGNCHPO, CLINVAR]
[ABCA1 gene, Neuropathy, VCP gene, Alzheimer's Disease]	[associated_with, associated_with, inverse_gene_associated_with_disease_or_phenotype]	[HGNCHPO, HGNCHPO, CLINVAR]
[ABCA1 gene, Neuropathy, Nervous system structure, Alzheimer's Disease]	[associated_with, disease_has_associated_anatomic_site, is_associated_anatomic_site_of]	[HGNCHPO, NCI, NCI]
[ABCA1 gene, Neuropathy, Nervous system structure, Alzheimer's Disease]	[associated_with, disease_has_associated_anatomic_site, disease_has_associated_anatomic_site]	[HGNCHPO, NCI, NCI]
[ABCA1 gene, Neuropathy, GRN gene, Alzheimer's Disease]	[associated_with, inverse_associated_with, gene_associated_with_disease_or_phenotype]	[HGNCHPO, HGNCHPO, CLINVAR]
[ABCA1 gene, Neuropathy, GRN gene, Alzheimer's Disease]	[associated_with, inverse_associated_with, inverse_gene_associated_with_disease_or_phenotype]	[HGNCHPO, HGNCHPO, CLINVAR]
[ABCA1 gene, Neuropathy, MAPT gene, Alzheimer's Disease]	[associated_with, inverse_associated_with, gene_associated_with_disease_or_phenotype]	[HGNCHPO, HGNCHPO, CLINVAR]
[ABCA1 gene, Neuropathy, MAPT gene, Alzheimer's Disease]	[associated_with, inverse_associated_with, inverse_gene_associated_with_disease_or_phenotype]	[HGNCHPO, HGNCHPO, CLINVAR]
[ABCA1 gene, Neuropathy, PSEN1 gene, Alzheimer's Disease]	[associated_with, inverse_associated_with, gene_associated_with_disease]	[HGNCHPO, HGNCHPO, NCI]
[ABCA1 gene, Neuropathy, PSEN1 gene, Alzheimer's Disease]	[associated_with, inverse_associated_with, gene_associated_with_disease_or_phenotype]	[HGNCHPO, HGNCHPO, CLINVAR]
[ABCA1 gene, Neuropathy, PSEN1 gene, Alzheimer's Disease]	[associated_with, inverse_associated_with, inverse_gene_associated_with_disease_or_phenotype]	[HGNCHPO, HGNCHPO, CLINVAR]
[ABCA1 gene, Neuropathy, PSEN1 gene, Alzheimer's Disease]	[associated_with, inverse_associated_with, disease_has_associated_gene]	[HGNCHPO, HGNCHPO, NCI]
[ABCA1 gene, Neuropathy, VCP gene, Alzheimer's Disease]	[associated_with, inverse_associated_with, gene_associated_with_disease_or_phenotype]	[HGNCHPO, HGNCHPO, CLINVAR]
[ABCA1 gene, Neuropathy, VCP gene, Alzheimer's Disease]	[associated_with, inverse_associated_with, inverse_gene_associated_with_disease_or_phenotype]	[HGNCHPO, HGNCHPO, CLINVAR]
[ABCA1 gene, Platelet count outside reference range, APOE gene, Alzheimer's Disease]	[associated_with, associated_with, gene_associated_with_disease]	[HGNCHPO, HGNCHPO, NCI]

## TREM2 3-hop paths

### Query:
- PNG query:
```cypher
MATCH path = (g:Gene {node_label:'TREM2 gene'})-[*1..3]-(d:Disease {node_label:"Alzheimer's Disease"})
RETURN path
LIMIT 25;
```
- Summary output query:
```cypher
MATCH path = (g:Gene {node_label:'TREM2 gene'})-[*1..3]-(d:Disease {node_label:"Alzheimer's Disease"})
WITH path, nodes(path) AS ns, relationships(path) AS rs
RETURN
  [n IN ns | n.node_label] AS nodes,
  [r IN rs | type(r)] AS relationships,
  [r IN rs | r.SAB] AS sources
LIMIT 25;
```

### Output: 

PNG: Output is in images_neo4j_queries/TREM2 3-hop_paths.png

nodes	relationships	sources
[TREM2 gene, Fatigue, central nervous system, Alzheimer's Disease]	[associated_with, PAR, PAR]	[HGNCHPO, OMIM, OMIM]
[TREM2 gene, Fatigue, central nervous system, Alzheimer's Disease]	[associated_with, PAR, CHD]	[HGNCHPO, OMIM, OMIM]
[TREM2 gene, Fatigue, central nervous system, Alzheimer's Disease]	[associated_with, PAR, is_associated_anatomic_site_of]	[HGNCHPO, OMIM, NCI]
[TREM2 gene, Fatigue, central nervous system, Alzheimer's Disease]	[associated_with, PAR, disease_has_associated_anatomic_site]	[HGNCHPO, OMIM, NCI]
[TREM2 gene, Fatigue, central nervous system, Alzheimer's Disease]	[associated_with, CHD, PAR]	[HGNCHPO, OMIM, OMIM]
[TREM2 gene, Fatigue, central nervous system, Alzheimer's Disease]	[associated_with, CHD, CHD]	[HGNCHPO, OMIM, OMIM]
[TREM2 gene, Fatigue, central nervous system, Alzheimer's Disease]	[associated_with, CHD, is_associated_anatomic_site_of]	[HGNCHPO, OMIM, NCI]
[TREM2 gene, Fatigue, central nervous system, Alzheimer's Disease]	[associated_with, CHD, disease_has_associated_anatomic_site]	[HGNCHPO, OMIM, NCI]
[TREM2 gene, Fatigue, VCP gene, Alzheimer's Disease]	[associated_with, associated_with, gene_associated_with_disease_or_phenotype]	[HGNCHPO, HGNCHPO, CLINVAR]
[TREM2 gene, Fatigue, VCP gene, Alzheimer's Disease]	[associated_with, associated_with, inverse_gene_associated_with_disease_or_phenotype]	[HGNCHPO, HGNCHPO, CLINVAR]
[TREM2 gene, Fatigue, VCP gene, Alzheimer's Disease]	[associated_with, inverse_associated_with, gene_associated_with_disease_or_phenotype]	[HGNCHPO, HGNCHPO, CLINVAR]
[TREM2 gene, Fatigue, VCP gene, Alzheimer's Disease]	[associated_with, inverse_associated_with, inverse_gene_associated_with_disease_or_phenotype]	[HGNCHPO, HGNCHPO, CLINVAR]
[TREM2 gene, Autosomal recessive inheritance, GRN gene, Alzheimer's Disease]	[associated_with, associated_with, gene_associated_with_disease_or_phenotype]	[HGNCHPO, HGNCHPO, CLINVAR]
[TREM2 gene, Autosomal recessive inheritance, GRN gene, Alzheimer's Disease]	[associated_with, associated_with, inverse_gene_associated_with_disease_or_phenotype]	[HGNCHPO, HGNCHPO, CLINVAR]
[TREM2 gene, Autosomal recessive inheritance, APOE gene, Alzheimer's Disease]	[associated_with, associated_with, gene_associated_with_disease]	[HGNCHPO, HGNCHPO, NCI]
[TREM2 gene, Autosomal recessive inheritance, APOE gene, Alzheimer's Disease]	[associated_with, associated_with, disease_has_associated_gene]	[HGNCHPO, HGNCHPO, NCI]
[TREM2 gene, Autosomal recessive inheritance, CSF1R gene, Alzheimer's Disease]	[associated_with, associated_with, gene_associated_with_disease_or_phenotype]	[HGNCHPO, HGNCHPO, CLINVAR]
[TREM2 gene, Autosomal recessive inheritance, CSF1R gene, Alzheimer's Disease]	[associated_with, associated_with, inverse_gene_associated_with_disease_or_phenotype]	[HGNCHPO, HGNCHPO, CLINVAR]
[TREM2 gene, Autosomal recessive inheritance, MPO gene, Alzheimer's Disease]	[associated_with, associated_with, gene_associated_with_disease_or_phenotype]	[HGNCHPO, HGNCHPO, CLINVAR]
[TREM2 gene, Autosomal recessive inheritance, MPO gene, Alzheimer's Disease]	[associated_with, associated_with, inverse_gene_associated_with_disease_or_phenotype]	[HGNCHPO, HGNCHPO, CLINVAR]
[TREM2 gene, Autosomal recessive inheritance, MAPT gene, Alzheimer's Disease]	[associated_with, associated_with, gene_associated_with_disease_or_phenotype]	[HGNCHPO, HGNCHPO, CLINVAR]
[TREM2 gene, Autosomal recessive inheritance, MAPT gene, Alzheimer's Disease]	[associated_with, associated_with, inverse_gene_associated_with_disease_or_phenotype]	[HGNCHPO, HGNCHPO, CLINVAR]
[TREM2 gene, Autosomal recessive inheritance, GRN gene, Alzheimer's Disease]	[associated_with, inverse_associated_with, gene_associated_with_disease_or_phenotype]	[HGNCHPO, HGNCHPO, CLINVAR]
[TREM2 gene, Autosomal recessive inheritance, GRN gene, Alzheimer's Disease]	[associated_with, inverse_associated_with, inverse_gene_associated_with_disease_or_phenotype]	[HGNCHPO, HGNCHPO, CLINVAR]
[TREM2 gene, Autosomal recessive inheritance, APOE gene, Alzheimer's Disease]	[associated_with, inverse_associated_with, gene_associated_with_disease]	[HGNCHPO, HGNCHPO, NCI]

## Results: 
| Gene | Direct AD link? | 2-hop AD path? | 3-hop AD path? | Interpretation |
|---|---|---|---|---|
| APOE | Yes | Yes, via Amyloidosis | Yes | Strong CondensedKG support |
| ABCA1 | No | No | Yes, weak/indirect | Better as Reactome lipid/cholesterol context |
| TREM2 | No | No | Yes, weak/indirect | Better as Reactome immune/microglia context |

# Neighbor node analysis for APOE, ABCA1, and TREM2

## APOE top neighbors

### Query:
```cypher
MATCH (g:Gene {node_label:'APOE gene'})-[r]-(n)
WHERE any(label IN labels(n) WHERE label IN ['Disease','EFO','GO','Protein','Compound'])
  AND NOT toLower(n.node_label) CONTAINS 'measurement'
  AND NOT toLower(n.node_label) CONTAINS 'procedure'
  AND NOT toLower(n.node_label) CONTAINS 'determination'
RETURN 
  labels(n) AS neighbor_type,
  n.node_label AS neighbor,
  type(r) AS relationship,
  r.SAB AS source
LIMIT 50;
```

### Output:

neighbor_type	neighbor	relationship	source
[Disease]	Apolipoprotein gene	isa	SNOMEDCT_US
[Disease]	Apolipoprotein E	RO	MTH
[Disease]	Apolipoprotein E	RO	MTH
[Disease]	Apolipoprotein gene	inverse_isa	SNOMEDCT_US
[EFO]	Amyloidosis	associated_with	HGNCHPO
[Disease]	Stricture of artery	associated_with	HGNCHPO
[Disease]	Urogenital Diseases	associated_with	HGNCHPO
[Disease]	Spastic tetraparesis	associated_with	HGNCHPO
[Disease]	Skin Abnormalities	associated_with	HGNCHPO
[Disease]	Seizures	associated_with	HGNCHPO
[Disease]	Babinski Reflex	associated_with	HGNCHPO
[Disease]	Cerebral Amyloid Angiopathy	associated_with	HGNCHPO
[Disease]	Agnosia	associated_with	HGNCHPO
[Disease]	Bleeding tendency	associated_with	HGNCHPO
[Disease]	Dystonia Disorders	associated_with	HGNCHPO
[Disease]	Fluid imbalance	associated_with	HGNCHPO
[Disease]	Muscle Spasticity	associated_with	HGNCHPO
[Disease]	Involuntary Movements	associated_with	HGNCHPO
[Disease]	Reflex, Abnormal	associated_with	HGNCHPO
[Disease]	Diabetes Mellitus	associated_with	HGNCHPO
[Disease]	Hypolipoproteinemias	associated_with	HGNCHPO
[Disease]	Skin Diseases, Vascular	associated_with	HGNCHPO
[Disease]	Splenomegaly	associated_with	HGNCHPO
[Disease]	Lesion of spleen	associated_with	HGNCHPO
[Disease]	Corneal Opacity	associated_with	HGNCHPO
[Disease]	Pyramidal sign	associated_with	HGNCHPO
[Disease]	Secondary Parkinson Disease	associated_with	HGNCHPO
[Disease]	Glucose Intolerance (disease)	associated_with	HGNCHPO
[Disease]	Liver Cirrhosis	associated_with	HGNCHPO
[Disease]	Inflamed joint	associated_with	HGNCHPO
[Disease]	Forgetful	associated_with	HGNCHPO
[Disease]	Peripheral arterial occlusive disease	associated_with	HGNCHPO
[Disease]	Movement Disorders	associated_with	HGNCHPO
[Disease]	Sea-Blue Histiocyte Syndrome	associated_with	HGNCHPO
[Disease]	Amyloidosis	associated_with	HGNCHPO
[Disease]	Low Vision	associated_with	HGNCHPO
[Disease]	Hyperreflexia	associated_with	HGNCHPO
[Disease]	Xanthoma tendinosum	associated_with	HGNCHPO
[Disease]	Atrophic retina	associated_with	HGNCHPO
[Disease]	Retinal Degeneration	associated_with	HGNCHPO
[Disease]	Clinical course	associated_with	HGNCHPO
[Disease]	Pancreatitis, Acute	associated_with	HGNCHPO
[Disease]	Fatty Liver	associated_with	HGNCHPO
[Disease]	Eye Hemorrhage	associated_with	HGNCHPO
[Disease]	Visceromegaly	associated_with	HGNCHPO
[Disease]	Renal Insufficiency	associated_with	HGNCHPO
[Disease]	Personality Change	associated_with	HGNCHPO
[Disease]	Autosomal domi t inheritance	associated_with	HGNCHPO
[Disease]	Hyperlipoproteinemias	associated_with	HGNCHPO
[Disease]	Melanoderma (disorder)	associated_with	HGNCHPO

## ABCA1 top neighbors

### Query:
```cypher
MATCH (g:Gene {node_label:'ABCA1 gene'})-[r]-(n)
WHERE any(label IN labels(n) WHERE label IN ['Disease','EFO','GO','Protein','Compound'])
  AND NOT toLower(n.node_label) CONTAINS 'measurement'
  AND NOT toLower(n.node_label) CONTAINS 'procedure'
  AND NOT toLower(n.node_label) CONTAINS 'determination'
RETURN 
  labels(n) AS neighbor_type,
  n.node_label AS neighbor,
  type(r) AS relationship,
  r.SAB AS source
LIMIT 50;
```

### Output:
neighbor_type	neighbor	relationship	source
[Disease]	Familial HDL deficiency	RO	MTH
[Disease]	Familial HDL deficiency	RO	MTH
[EFO]	Neuropathy	associated_with	HGNCHPO
[Disease]	Platelet count outside reference range	associated_with	HGNCHPO
[Disease]	Reflex, Abnormal	associated_with	HGNCHPO
[Disease]	Abdominal discomfort	associated_with	HGNCHPO
[Disease]	Nail Diseases	associated_with	HGNCHPO
[Disease]	Facial Paresis	associated_with	HGNCHPO
[Disease]	Reduced sensation of skin	associated_with	HGNCHPO
[Disease]	Corneal Opacity	associated_with	HGNCHPO
[Disease]	Dystrophia unguium	associated_with	HGNCHPO
[Disease]	Pain	associated_with	HGNCHPO
[Disease]	Left Ventricular Hypertrophy	associated_with	HGNCHPO
[Disease]	Serum triglycerides increased	associated_with	HGNCHPO
[Disease]	Hepatosplenomegaly	associated_with	HGNCHPO
[Disease]	Peripheral Nervous System Diseases	associated_with	HGNCHPO
[Disease]	Muscle Weakness	associated_with	HGNCHPO
[Disease]	Lesion of spinal cord	associated_with	HGNCHPO
[Disease]	Ectropion	associated_with	HGNCHPO
[Disease]	Ventricular hypertrophy	associated_with	HGNCHPO
[Disease]	Autosomal domi t inheritance	associated_with	HGNCHPO
[Disease]	Coronary Arteriosclerosis	associated_with	HGNCHPO
[Disease]	Stricture of artery	associated_with	HGNCHPO
[Disease]	Hepatomegaly	associated_with	HGNCHPO
[Disease]	Visceromegaly	associated_with	HGNCHPO
[Disease]	Lymphatic Diseases	associated_with	HGNCHPO
[Disease]	Axonal neuropathy	associated_with	HGNCHPO
[Disease]	Thrombasthenia	associated_with	HGNCHPO
[Disease]	Carotid Stenosis	associated_with	HGNCHPO
[Disease]	Splenomegaly	associated_with	HGNCHPO
[Disease]	Left ventricular abnormality	associated_with	HGNCHPO
[Disease]	Low Vision	associated_with	HGNCHPO
[Disease]	Arteriosclerosis	associated_with	HGNCHPO
[Disease]	Cicatricial ectropion	associated_with	HGNCHPO
[Disease]	Lesion of spleen	associated_with	HGNCHPO
[Disease]	Electromyogram abnormal	associated_with	HGNCHPO
[Disease]	Atherosclerosis	associated_with	HGNCHPO
[Disease]	Hypocholesterolemia	associated_with	HGNCHPO
[Disease]	Skin Abnormalities	associated_with	HGNCHPO
[Disease]	Hydrosyringomyelia	associated_with	HGNCHPO
[Disease]	Myocardial Infarction	associated_with	HGNCHPO
[Disease]	Disorder of face	associated_with	HGNCHPO
[Disease]	Coronary Stenosis	associated_with	HGNCHPO
[Disease]	Congenital heart disease	associated_with	HGNCHPO
[Disease]	Decreased tendon reflex	associated_with	HGNCHPO
[Disease]	Corneal stromal opacities	associated_with	HGNCHPO
[Disease]	Abnormality of red blood cells	associated_with	HGNCHPO
[Disease]	Hypolipoproteinemias	associated_with	HGNCHPO
[Disease]	Blurred vision	associated_with	HGNCHPO
[Disease]	Autosomal recessive inheritance	associated_with	HGNCHPO

## TREM2 top neighbors

### Query:
```cypher
MATCH (g:Gene {node_label:'TREM2 gene'})-[r]-(n)
WHERE any(label IN labels(n) WHERE label IN ['Disease','EFO','GO','Protein','Compound'])
  AND NOT toLower(n.node_label) CONTAINS 'measurement'
  AND NOT toLower(n.node_label) CONTAINS 'procedure'
  AND NOT toLower(n.node_label) CONTAINS 'determination'
RETURN 
  labels(n) AS neighbor_type,
  n.node_label AS neighbor,
  type(r) AS relationship,
  r.SAB AS source
LIMIT 50;
```

### Output:
neighbor_type	neighbor	relationship	source
[EFO]	Hereditary Chorea	associated_with	HGNCHPO
[Disease]	Fatigue	associated_with	HGNCHPO
[Disease]	Autosomal recessive inheritance	associated_with	HGNCHPO
[Disease]	Oculomotor apraxia	associated_with	HGNCHPO
[Disease]	White blood cell abnormality	associated_with	HGNCHPO
[Disease]	Imaging of brain abnormal	associated_with	HGNCHPO
[Disease]	Abulia	associated_with	HGNCHPO
[Disease]	Hyperreflexia	associated_with	HGNCHPO
[Disease]	Frontotemporal dementia	associated_with	HGNCHPO
[Disease]	Dysthymic Disorder	associated_with	HGNCHPO
[Disease]	Mouth Abnormalities	associated_with	HGNCHPO
[Disease]	Pathological fracture	associated_with	HGNCHPO
[Disease]	Movement Disorders	associated_with	HGNCHPO
[Disease]	Hydrocephalus	associated_with	HGNCHPO
[Disease]	Finger Agnosia	associated_with	HGNCHPO
[Disease]	Compulsive hoarding	associated_with	HGNCHPO
[Disease]	Leukoencephalopathy	associated_with	HGNCHPO
[Disease]	Osteopenia	associated_with	HGNCHPO
[Disease]	Mutism	associated_with	HGNCHPO
[Disease]	Apraxias	associated_with	HGNCHPO
[Disease]	Mood swings	associated_with	HGNCHPO
[Disease]	Malig t Neoplasms	associated_with	HGNCHPO
[Disease]	Personality Change	associated_with	HGNCHPO
[Disease]	Nausea and vomiting	associated_with	HGNCHPO
[Disease]	Cerebral calcification	associated_with	HGNCHPO
[Disease]	Agnosia	associated_with	HGNCHPO
[Disease]	Amyotrophic Lateral Sclerosis	associated_with	HGNCHPO
[Disease]	Tonic - clonic seizures	associated_with	HGNCHPO
[Disease]	Stereotyped Behavior	associated_with	HGNCHPO
[Disease]	Muscle Spasticity	associated_with	HGNCHPO
[Disease]	Extrapyramidal Disorders	associated_with	HGNCHPO
[Disease]	Pyramidal sign	associated_with	HGNCHPO
[Disease]	Perseveration	associated_with	HGNCHPO
[Disease]	Dysgraphia	associated_with	HGNCHPO
[Disease]	Hematologic Neoplasms	associated_with	HGNCHPO
[Disease]	Respiratory Failure	associated_with	HGNCHPO
[Disease]	Calcification of basal ganglia	associated_with	HGNCHPO
[Disease]	Spasm	associated_with	HGNCHPO
[Disease]	Inappropriate behavior	associated_with	HGNCHPO
[Disease]	Bone Cysts	associated_with	HGNCHPO
[Disease]	Neurofibrillary degeneration (morphologic abnormality)	associated_with	HGNCHPO
[Disease]	Pain	associated_with	HGNCHPO
[Disease]	Disturbance of consciousness	associated_with	HGNCHPO
[Disease]	Reflex, Abnormal	associated_with	HGNCHPO
[Disease]	Obsessive compulsive behavior	associated_with	HGNCHPO
[Disease]	Babinski Reflex	associated_with	HGNCHPO
[Disease]	Seizures	associated_with	HGNCHPO
[Disease]	Degenerative brain disorder	associated_with	HGNCHPO
[Disease]	Muscle Weakness	associated_with	HGNCHPO
[Disease]	Secondary Parkinson Disease	associated_with	HGNCHPO

## Result:

## CondensedKG neighbor summary

| Gene | Useful KG neighbors | Interpretation |
|---|---|---|
| APOE | Amyloidosis, Cerebral Amyloid Angiopathy, Forgetful, Hyperlipoproteinemias | Strong Alzheimer’s/amyloid/lipid-related KG support |
| ABCA1 | Familial HDL deficiency, Hypocholesterolemia, Atherosclerosis, Coronary arteriosclerosis, Neuropathy | Better supports lipid/cholesterol/vascular biology than direct AD |
| TREM2 | Frontotemporal dementia, Leukoencephalopathy, Degenerative brain disorder, Neurofibrillary degeneration, brain imaging abnormal | Better supports neurodegeneration/brain phenotype context than direct AD |

