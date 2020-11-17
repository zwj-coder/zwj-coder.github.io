# Timestamp-Based Protocols

With each transaction $T_i$ in the system, we choose a unique fixed timestamp.  This timestamp is assigned by the database system before the transaction $T_i$ starts execution.

There are two simple methods to impletement this timestamp

1. Use the value of system clock
2. Use a logical counter

The timestamps of the transactions determine the serializability order. Thus is $TS(T_i) < TS(T_j)$, then the system ensure that the produced shedule is equivalent to a serial schedule in which transaction $T_i$ appear before the transaction $T_j$

## Implementation of timestamp ordering

we associate two timestamp values with each data item $Q$

1. $W$-$timestamp(Q)$  denotes the largest timestamp of any transaction that execute Write(Q)  successfully
2. $R$-$timestamp(Q)$ denotes the largest timestamp of any transaction that execute Read(Q) successfully

The timestamp ordering protocol ensure that any conflicting read and write operations are executed in timestamp order.

1. Suppose that transaction $T_i$ issues read(Q).

   a. If $TS(T_i) <$ W-timestamp(Q): 假定之前最大执行写操作的事务是$T_j$, 那么$TS(T_i) < TS(T_j)$, 但是现在要执行一个之前应该执行的读操作，违反timestamp顺序。所以这个读操作请求被拒绝，$T_i$执行roll back操作。

   b. If $TS(T_i) >=$ W-timestamp(Q): 同理，这说明$TS(T_i) > TS(T_j)$, 所以现在执行的读操作符合timestamp ordering，同时更新对应的R-timestamp(Q).

2. Suppose that transaction $T_i$ issues write(Q).

   a. If $TS(T_i) <$ R-timestamp(Q), 同理，拒绝并rollback $T_i$

   b. If $TS(T_i) <$ W-timestamp(Q), 同理，拒绝并rollback $T_i$

   c. 否则执行写操作，并更新W-timestamp(Q)为$TS(T_i)$的值
