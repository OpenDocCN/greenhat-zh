## 程序 13：程序有点问题

为什么这个程序在某些数值上会失败？此外，这个程序除了它旨在说明的问题外，还存在一个错误。另一个问题在哪里？

```
  1 /************************************************
  2  * Billing -- Print out how much we owe         *
  3  *      customers or they owe us.               *
  4  ************************************************/
  5 #include <iostream>
  6
  7 // Number of pennies in a dollar
  8 const int DOLLAR = 100;
  9
 10 /************************************************
 11  * billing -- do the billing.                   *
 12  *      If the customer owes us money           *
 13  *              -- output debt.                 *
 14  *      If we owe more than $100                *
 15  *              -- output credit.               *
 16  *      Between $0 and $100 just ignore the     *
 17  *              account.                        *
 18  ************************************************/
 19 int billing(
 20     // Current balance (in cents)
 21     const int balance
 22 ) {
 23     if (balance < 0)
 24         if (balance < - (100*DOLLAR))
 25             std::cout << "Credit " << -balance << endl;
 26     else
 27         std::cout << "Debt " << balance << endl;
 28
 29     return (0);
 30 }
 31
 32 int main()
 33 {
 34     /* Test code */
 35     billing(50);
 36     billing(-10);
 37     return (0);
 38 }

```

（下一提示 44。答案 31。）
