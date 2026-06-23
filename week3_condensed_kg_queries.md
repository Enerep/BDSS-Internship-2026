# Pull compound neighbors

### Query
```cypher
MATCH (g:Gene)-[r]-(n:Compound)
WHERE g.node_label IN ['NR1H3 gene', 'NR1H2 gene', 'TYROBP gene']
RETURN g.node_label AS gene,
       n.node_label AS compound,
       n.node_code AS compound_code,
       n.CUI AS cui,
       type(r) AS relationship,
       r.SAB AS source
ORDER BY g.node_label
```

### Output
`result is in kg_output_csv/target_compounds_raw.csv` 
