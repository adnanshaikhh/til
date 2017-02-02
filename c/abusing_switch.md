The ```switch``` statement is a conceptual jump-table, this means if ```x==CMD1```
it will bypass the default and jump straight to case label ```CMD1```
Thus, we can do this

```
void process_message(int x)
{
    switch (x) {
        default:
            if (valid_command_message(x)) {
            case CMD1: case CMD2: case CMD3: case CMD4:
                process_command_msg(x);
                break;
            }
            else if (valid_status_message(x)) {
            case STATUS1: case STATUS2: case STATUS3:
                process_status_msg(x);
                break;
            }
            else
                report_error(x);
            break;
    }
}
```

And for the cherry on the top, the else statement (in this context) can act as
the equivalent of a break statement. We can, therefore, remove all the break
statements (thus also allowing us to remove the need for all the blocks)!

```
void process_message(int x)
{
    switch (x)
        default:
            if (valid_command_message(x))
            case CMD1: case CMD2: case CMD3: case CMD4:
                process_command_msg(x);
            else if (valid_status_message(x))
            case STATUS1: case STATUS2: case STATUS3:
                process_status_msg(x);
            else
                report_error(x);
}


```
