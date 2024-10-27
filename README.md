sqlite but with only prepared statements

```ahk
#Include %A_LineFile%/../sqlite.ahk

db:=sqlite(":memory:", A_LineFile "\..\sqlite3.dll") ;path to sqlite3.dll is needed
; ":memory:" is temporary, use a filename for persistent storage

; normal usage:
; db:=sqlite("db.sqlite", A_LineFile "\..\sqlite3.dll")

db.prepare("
(
CREATE TABLE debts (
   DebtName TEXT,
   Amount TEXT,
   CreatedDate TEXT
`)
)").all() ;create table using prepared statement

view_db() {
    static select_statement := db.prepare("select DebtName,Amount,CreatedDate from debts")

    rows:=select_statement.all() ;Array
    for row in rows {
        OutputDebug row.DebtName ": " row.Amount "`n"

        ; works too:
        ; OutputDebug row["DebtName"] ": " row["Amount"] "`n"

        ; works too: (useful for when you don't know the name of the column)
        ; OutputDebug row[1] ": " row[2] "`n"

        ; for k,v in row { ; Enumerator gives you column names (and values)
        ;     OutputDebug k ": " v "`n"
        ; }
    }
}

;Array mode: indexed params ("?" , "?NNN")
db.prepare("insert into debts (DebtName,Amount,CreatedDate) VALUES (?,?,?)").all("Medical Bill",2000,"2024-08-25")
db.prepare("update debts set Amount=? where DebtName=?").all(1500,"Medical Bill")
view_db()

;Object mode: named params (":AAA", "@AAA", "$AAA")
db.prepare("update debts set Amount=$Amount where DebtName=$DebtName").all({Amount:1000,DebtName:"Medical Bill"}) ;using {} Object
view_db()

;Object mode: named params (":AAA", "@AAA", "$AAA")
db.prepare("update debts set Amount=:Amount where DebtName=:DebtName").all(Map("Amount",500,"DebtName","Medical Bill")) ;using Map() works too
view_db()
```

.loadExtension() is not supported but will be added on request, and more

better error messages is being worked on, any recommendation/error report is appreciated