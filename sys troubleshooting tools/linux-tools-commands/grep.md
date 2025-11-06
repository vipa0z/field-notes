
search for db methods/functions
- `r` → recursive (search inside all subfolders under `flask/`)
    
- `n` → show line numbers
    
- `w` → match whole word (optional; you can omit if you want partial matches)
    
- `flask/` → target folder
    
- `"pattern"` → the pattern to search for (quoted to prevent shell interpretation)
`grep -rnw "db_conf()" flask/`
