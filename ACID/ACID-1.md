# ACID

# Atomicity

**One of the four atomic properties **

*it defines any type of databse systems*

> all queries in a transaction should succeed, 


- All queries in a transaction must succeed
  - back when atomicity principle was invented, at that point in time, an atom was supposed to be the smallest entity it couldn't be break down into smaller entities
- if one query fails, than all prior successfull queries in the transaction should rollback
- if the database went down prior to a commit of a transaction, all the successfull queries in the transaction should rollback

> now some things to wonder about databases what it is actually doing like when a transaction fails, does it goes to the disk and erase all the committed transactions... ?


##### Lets look at an example


| Account_id    | Balance       |
| ------------- |:-------------:|
| 1             | $1000         | 
| 2             | $500          |   

1. we have a account balance table where there are two accounts with some balance , we are going to move $100 from account 1 to 2
2. for that we are going to start a transaction
    
    Transaction BEGIN
         |   
         | SELECT BALANCE WHERE ACCOUNT_ID=1 
         | BALANCE > 100
         |                 MINUS 100 FROM 1000 FOR ACCOUNT_ID = 1
         |                 ADD 100 TO ACCOUNT_ID = 2
         |
   Transaction END
   
3. After the transaction ended the table looks somewhat like this

| Account_id    | Balance       |
| ------------- |:-------------:|
| 1             | $900          | 
| 2             | $600          |  


4. Now suppose while the deduction was going on, either one of the query fails or the data base crashes, the **ROLLBACK** has to happen
5. now suppose after crashing we decided to restart the machine, then what... well, after we restarted the machine the first account has been debited but the second account has not been credited yet.
6. This is really bad as we just *lost the data* , and the information is **inconsistent ** 
7. See now, an **Atom Transaction** is a transaction that will rollback all queries if one or more queries fails.
8. rollback can take more than 1 hour for long transactions like for sql server.
9. Now while rollback all the shit that was there has to be cleaned like above all the queries return, which in return makes the cpu and the memories getting hammered.
10. so longer transaction can be long
11. atomicity says that it has to be single operation that can not be split
12. if we have a transaction having 100 queris all the query has to be success, if any one of the fails ROLLBACK has to happen.

     


# Isolation
- this one is really the important property of acid. 
- lets think to answer this following question -> 

> Can my inflight transaction see the changes made by other transaction.

- lets jot down some points
- concurrency 
- multiple transactions fighting each other to read or write over some allocated resources (same piece of data)
- that is where the isolation cames in , and we ask us the above question again.
- now conside a transaction that does a commit, so another transaction if driving on the same resource, should see that commit or not, well it depends, do you want to see that change?
- there is no right and wrong, this result in something called **Read Phenomenas**, this is really undesirable and should be avoided, and this is one of the nastiest thing to debug.
- to solve the above issue **Isolation Levels** were introduced to solve the above undesirable read phenomena


#### *Isolation Read phenomenas -> *

## Dirty Reads
   - when an inflight transaction that is currently running, it read something, that some other transaction has wrote and didnt has commited something yet. so there is a chances that this change that we read could be rollbacked or it could be committed, or the database crashed.
   - so dirty read is nothing but that when you wrote something that is not **fully flushed, not fully committed**


## Non-Repeated Reads
   - Suppose you are in a transaction and then you read a value, then again in the same transaction you read that same value again. now why would you ever do the same thing twice? ... this is not required, but this happens all the time.
   - like different queries resulting in the same value.
     - like a query read some value
     - then another query is doing sum on those values, in this case again this query will read all the values that the previous query already read.
   - now what if we read something at time t1, than again at time t2, but from time t1 -> t2 the value changes. then what fucktard! 
   - that is called **non-repeatable**. ... some people call this undesirable


## Phantom Reads
   - these are things that we cant read because these are *not existing yet*, example are *range queries* .
   - like we have a **start** and **end**, we used that we got some values, now later on in the **same range**  a new row is added that satisifies the above range, now this is not non-repeated, 
   - now if we do another select,, then we are going to get that *phantom new row* and then you read, and things that can go really wrong.
   - like first time you got something and then the second something entirely else.
   - this seems like non-repeated reads but its not, because we didn't read it actually, we can't catch it

## Lost updates
  - i wrote something but before i commit, i tried to read what i wrote, well this is very common , because we are in our own bubble that is we can read something that we wrote. 