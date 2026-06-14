# Node Lookup

## Find NR1H3 / LXR nodes

### Query

```cypher
MATCH (n)
WHERE toLower(n.node_label) CONTAINS 'nr1h3'
   OR toLower(n.node_label) CONTAINS 'liver x receptor'
   OR toLower(n.node_label) CONTAINS 'lxr'
RETURN labels(n) AS labels,
       n.node_label AS node,
       n.node_code AS node_code,
       n.CUI AS cui,
       n.SAB AS source
LIMIT 50;
```

### Output:
- ["Gene"] "NR1H3 gene" "7966" "C1417824" "HGNC"
- ["Protein"] "Oxysterols receptor LXR-beta" "P55055" "UNIPROTKB:P55055 CUI" "UNIPROTKB"
- ["Protein"] "Oxysterols receptor LXR-alpha" "Q13133" "UNIPROTKB:Q13133 CUI" "UNIPROTKB"

## Find TYROBP/DAP12

### Query
```cypher
MATCH (n)
WHERE toLower(n.node_label) CONTAINS 'tyrobp'
   OR toLower(n.node_label) CONTAINS 'dap12'
RETURN labels(n) AS labels,
       n.node_label AS node,
       n.node_code AS node_code,
       n.CUI AS cui,
       n.SAB AS source
LIMIT 50;
```

### Output: 
- ["Gene"] "TYROBP gene" "12449" "C1336692" "HGNC"


# Top neightbors:

## NR1H3 neighbors:

### Query:
```cypher
MATCH (g:Gene {node_label:'NR1H3 gene'})-[r]-(n)
WHERE any(label IN labels(n) WHERE label IN ['Disease','EFO','GO','Protein','Compound','Gene'])
  AND NOT toLower(n.node_label) CONTAINS 'measurement'
  AND NOT toLower(n.node_label) CONTAINS 'procedure'
  AND NOT toLower(n.node_label) CONTAINS 'determination'
RETURN labels(n) AS neighbor_type,
       n.node_label AS neighbor,
       n.node_code AS node_code,
       n.CUI AS cui,
       type(r) AS relationship,
       r.SAB AS source
LIMIT 1000;
```
### Output: 
`Can be found in kg_output_csv/nr1h3_top_neighbors.csv`

## TYROBP neighbors:

### Query:
```cypher
MATCH (g:Gene {node_label:'TYROBP gene'})-[r]-(n)
WHERE any(label IN labels(n) WHERE label IN ['Disease','EFO','GO','Protein','Compound','Gene'])
  AND NOT toLower(n.node_label) CONTAINS 'measurement'
  AND NOT toLower(n.node_label) CONTAINS 'procedure'
  AND NOT toLower(n.node_label) CONTAINS 'determination'
RETURN labels(n) AS neighbor_type,
       n.node_label AS neighbor,
       n.node_code AS node_code,
       n.CUI AS cui,
       type(r) AS relationship,
       r.SAB AS source
LIMIT 1000;
```

### Output:
`Can be found in kg_output_csv/tyrobp_top_neighbors.csv`

# Short path from NR1H3 and TYROBP to APOE/ABCA1/TREM2/AD

## NR1H3 

### Query

```cypher
MATCH (start:Gene {node_label:'NR1H3 gene'})
MATCH (end)
WHERE end.node_label IN ['APOE gene','ABCA1 gene','TREM2 gene']
   OR (end:Disease AND end.node_label = "Alzheimer's Disease" AND end.node_code = '26929004')
MATCH p = shortestPath((start)-[*..4]-(end))
RETURN start.node_label AS start,
       end.node_label AS end,
       labels(end) AS end_type,
       length(p) AS path_length,
       [node IN nodes(p) | node.node_label] AS path_nodes,
       [rel IN relationships(p) | type(rel)] AS path_relationships
ORDER BY path_length;
```

### Output
`Can be found in kg_output_csv/nr1h3.csv`

## TYROBP

### Query

```cypher
MATCH (start:Gene {node_label:'TYROBP gene'})
MATCH (end)
WHERE end.node_label IN ['APOE gene','ABCA1 gene','TREM2 gene']
   OR (end:Disease AND end.node_label = "Alzheimer's Disease" AND end.node_code = '26929004')
MATCH p = shortestPath((start)-[*..4]-(end))
RETURN start.node_label AS start,
       end.node_label AS end,
       labels(end) AS end_type,
       length(p) AS path_length,
       [node IN nodes(p) | node.node_label] AS path_nodes,
       [rel IN relationships(p) | type(rel)] AS path_relationships
ORDER BY path_length;
```

### Output

`Can be found in kg_output_csv/tyrobp_shortest_path.csv`

# Other queries:

## Compare neighbor counts
### Query:
```cypher
MATCH (g:Gene)-[r]-(n)
WHERE g.node_label IN ['NR1H3 gene','TYROBP gene']
  AND any(label IN labels(n) WHERE label IN ['Disease','EFO','GO','Protein','Compound','Gene'])
  AND NOT toLower(n.node_label) CONTAINS 'measurement'
  AND NOT toLower(n.node_label) CONTAINS 'procedure'
  AND NOT toLower(n.node_label) CONTAINS 'determination'
RETURN g.node_label AS gene,
       labels(n) AS neighbor_type,
       count(*) AS count
ORDER BY gene, count DESC;
```
### Output:
`Can be found in kg_output_csv/neighbor_counts.csv`

## Relationship type counts

### Query:
```cypher
MATCH (g:Gene)-[r]-(n)
WHERE g.node_label IN ['NR1H3 gene','TYROBP gene']
  AND any(label IN labels(n) WHERE label IN ['Disease','EFO','GO','Protein','Compound','Gene'])
RETURN g.node_label AS gene,
       type(r) AS relationship,
       r.SAB AS source,
       count(*) AS count
ORDER BY gene, count DESC;
```

### Output:
`Can be found in kg_output_csv/relationship_type_counts.csv`

