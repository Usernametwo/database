# plsql中的包package

package是函数或者过程的集合

```sql
--program window -> package
--declare
CREATE OR REPLACE PACKAGE comm_package IS
    --public variables
    g_comm NUMBER := 10;
    --declare public procedure or function
    PROCEDURE reset_comm(p_comm IN NUMBER);
END comm_package;

--program window -> package
--define
CREATE OR REPLACE PACKAGE BODY comm_package IS
    -- private function
    FUNCTION validate_comm(p_comm IN NUMBER) RETURN BOOLEAN IS
        v_max_comm NUMBER;
    BEGIN
        v_max_comm := 10;
        reset_comm(p_comm);
        RETURN v_max_comm = 10;
    END validate_comm;
    --public procedure
    PROCEDURE reset_comm(p_comm IN NUMBER) IS
    BEGIN
        NULL;
    END reset_comm;

BEGIN
    --initialization
    NULL;
END comm_package;

```

定义package必须先声明再实现

package的向前声明特性，在 package body 中，一个函数中调用另一个函数，则另一个函数必须在前面先定义，或者那个函数是公有函数.

重载：一个package 中可以定义同名、不同参数的函数或过程

Package中的公共变量，不同的Session不会相互影响