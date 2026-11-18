Pipes |> ||> |>> ||>> $|> $||> ?|> ?||> ?|>> ?||>> ?$|> ?$||>
-------------------------------------------------------------

### Pipes |>

var |> fn1() |> fn2(5) |> fn3(5, 6)
===================================

fn3(fn2(fn1(var), 5), 5, 6)

### Fat Pipes ||>

var |> fn1() ||> fn2(5) ||> fn3(5, 6)
=====================================

const t = fn1(var) fn2(t, 5) fn3(t, 5, 6)

### Reverse Pipes |>> ||>>

var |> fn1() |>> fn2(5) |>> fn3(5, 6)
=====================================

fon3(5, 6, fn2(5, fn1(var))

var |> fn1() ||>> fn2(5) ||>> fn3(5, 6)
=======================================

const t = fn1(var) fn2(5, t) fn3(5, 6, t)

### Capture Pipes $|> $||>

users

```
|> user_filter_by_name(username)
$|> $.0 
$||> log("Attempted login: ", $.username)
|> login(password)
```

==

const user = user_filter_by_name(users, username).0; log("Attempted login: ", user.username) user_login(user, password)

### Optional Pipes ?|> ?||> ?|>> ?||>> ?$|> ?$||>

users

```
|> user_get_by_name(username)
?$||> log("Attempted login: ", $.username)
?|> login(password)
```

==

const user = user_get_by_name(users, username) if (user is not null) {

```
log("Attempted login: ", user.username)
user_login(user, password)
```

\}

### Fallback Pipes !|> !||>

value

```
!|> default_value()
|> continue_processing()
```

==

if (value is null) {

```
value = default_value()
```

\} continue_processing(value)

value

```
!||> log_missing_value()
|> continue_processing()
```

==

if (value is null) {

```
log_missing_value()
```

} else {

```
continue_processing(value)
```

\}
