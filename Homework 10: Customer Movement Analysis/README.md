# Customer Movement Analysis
This task is about analyzing movements of customers of a supermarket. The raw data comes in the size of 22 columns x 956K rows. I will use Google BigQuery for data preparation process and PowerBI to visualize and gain insight(s) from the result.
Customer movement analysis will be based on the following categories;
* New customers
* Repetitive customers
* Reactivated customers
* Churn

### 1. Prepare data via SQL
```SQL
select  SHOP_MONTH,
        CUST_CODE,
        TXN_M,
        TXN_LM,
        CUM_TXN,
        case when TXN_M = CUM_TXN and TXN_M = TXN_LM then 'NEW'
        when TXN_M != TXN_LM then 'REPEAT'
        when TXN_M is not null or TXN_M = TXN_LM then 'REACTIVATED'
        when (TXN_M is null or TXN_LM is null) and CUM_TXN is not null then 'CHURN'
        else 'NOT EXCLUDED'
        end as MOVEMENT_STATUS
from (
        select  aa.SHOP_MONTH,
                bb.CUST_CODE,
                cc.TXN_M,
                sum(cc.TXN_M) over (partition by bb.CUST_CODE order by aa.SHOP_MONTH rows between 1 preceding and current row) as TXN_LM,
                sum(cc.TXN_M) over (partition by bb.CUST_CODE order by aa.SHOP_MONTH) as CUM_TXN,
        from
                (select distinct cast(substr(cast(SHOP_DATE as string),1,6) as int64) as SHOP_MONTH from `secure-stone-272416.BADS7105.bads7105`) aa
        cross join
                (select distinct CUST_CODE from `secure-stone-272416.BADS7105.bads7105` where CUST_CODE is  not null) bb
        left join (
                select  cast(substr(cast(SHOP_DATE as string),1,6) as int64) as SHOP_MONTH,
                        CUST_CODE,
                        count(*) as TXN_M
                from    `secure-stone-272416.BADS7105.bads7105`
                where   CUST_CODE is  not null
                group by cast(substr(cast(SHOP_DATE as string),1,6) as int64),CUST_CODE
                ) cc
        on bb.CUST_CODE = cc.CUST_CODE and aa.SHOP_MONTH = cc.SHOP_MONTH
        order by bb.CUST_CODE, aa.SHOP_MONTH
)
where TXN_LM is not null
;
```

### 2. Visualize output by PowerBI
![Picture8](https://user-images.githubusercontent.com/59596996/122715835-b1cd9d80-d293-11eb-88f2-9a0459214a0e.jpg)

### 3. Gain insights
