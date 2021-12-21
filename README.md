# Non propagation of native options when running command in command

There are 2 commands `app:foo` and `app:bar`.

`app:foo` depends on `app:bar`, and run it with 2 options `--option1` and `--no-interaction`

## Scenarios

1. Running `bin/console app:bar` will ask you nothing.

```shell
$ bin/console app:bar


 [OK] app:bar success!


```

2. Running `bin/console app:bar --option1` will ask you

```shell
$ bin/console app:bar --option1

 Are you sure to continue? [yes]:
 >
```

3. Running `bin/console app:bar --option1 --no-interaction` will ask you nothing.

```shell
$ bin/console app:bar --option1 --no-interaction


 [OK] app:bar success!


```

4. Running `bin/console app:foo` or `bin/console app:foo --no-interaction` will always ask you

```shell
$ bin/console app:foo
Do something in app:foo command ...
Run app:bar ...

 Are you sure to continue? [yes]:
 >
```

```shell
$ bin/console app:foo --no-interaction
Do something in app:foo command ...
Run app:bar ...

 Are you sure to continue? [yes]:
 >
```

But the code calling `app:bar` is the following:

```php
$io->writeln('Run app:bar ...');
$this->getApplication()
    ->find('app:bar')
    ->run(new ArrayInput([
        '--option1' => true,
        '--no-interaction' => true,
    ]), $output)
;
```

We explicitly pass `--option1` (to trigger the interactive question) *and* `--no-interaction` to avoid any interaction.
As we saw in the 3rd example, this combination of options should not trigger interaction, but if we call `app:bar` from `app:foo` interaction is always triggered.
