# gotx
Go DB driver for transactionnal testing

## Ideas

- `func SandboxTx(db *sql.DB, fn func(txDB *sql.DB))`

```go
func Test_Stuff(t *test.Testing) {
  var db *sql.DB = anyMethodToProvideDB()
  gotx.SandboxTx(db, func(txDB *sql.DB) {
    // Do any CRUD you want in the db
  })
}

// Concrete example
func Test_Insertions(t *test.Testing) {
  // DB is empty
  gotx.SandboxTx(GetDB(), func(db *sql.DB) {
    db.Exec("INSERT INTO places (name) VALUES ('This is the place')")
    var category string
    err := db.QueryRow("SELECT category FROM places WHERE name = 'This is the place'").Scan(&category)
    if err != nil {
      t.Errorf("Select failed: %s", err)
    }
    if category != "default" {
      t.Errorf("Bad value for category (expecting: 'default', got %q)", category)
    }
    // DB is no more empty
  })
  // Global transaction rolled back
}

func Test_StuffWithTransactionInside(t *test.Testing) {
  // DB is empty
  gotx.SandboxTx(GetDB(), func(db *sql.DB) { // --> BEGIN;
    tx, _ := db.Begin()                      // --> SAVEPOINT gotx_23456789
    tx.Exec("INSERT INTO ...")               // --> INSERT INTO ...
    tx.Rollback()                            // --> ROLLBACK TO gotx_23456789
    // ...
    tx, _ := db.Begin()                      // --> SAVEPOINT gotx_22256742
    tx.Exec("INSERT INTO ...")               // --> INSERT INTO ...
    tx.Commit()                              // --> RELEASE SAVEPOINT gotx_22256742
  })                                         // --> ROLLBACK;
  // Global transaction rolled back
}
```
> Could also detect buggy things like commit called twice
