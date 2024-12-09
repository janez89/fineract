# Fineract Progressive Loan Embeddable Schedule Generator

## Build

- Generate Embeddable Progressive Schedule Generator Jar

    ```shell
        ./gradlew :fineract-progressive-loan-embeddable-schedule-generator:shadowJar
    ```

- Copy Jar from `fineract-progressive-loan-embeddable-schedule-generator/build/libs/fineract-progressive-loan-embeddable-schedule-generator-*-SNAPSHOT-all.jar` to Your class path.

## Dependencies

There is no extra dependency.


## Sample Application

Create a `Main.java` file with following sourcecode in `src` directory:

```java
/**
 * Licensed to the Apache Software Foundation (ASF) under one
 * or more contributor license agreements. See the NOTICE file
 * distributed with this work for additional information
 * regarding copyright ownership. The ASF licenses this file
 * to you under the Apache License, Version 2.0 (the
 * "License"); you may not use this file except in compliance
 * with the License. You may obtain a copy of the License at
 *
 * http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing,
 * software distributed under the License is distributed on an
 * "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
 * KIND, either express or implied. See the License for the
 * specific language governing permissions and limitations
 * under the License.
 */
import org.apache.fineract.organisation.monetary.data.CurrencyData;
import org.apache.fineract.portfolio.common.domain.DaysInMonthType;
import org.apache.fineract.portfolio.common.domain.DaysInYearType;
import org.apache.fineract.portfolio.loanaccount.loanschedule.data.LoanSchedulePlan;
import org.apache.fineract.portfolio.loanaccount.loanschedule.data.LoanSchedulePlanDisbursementPeriod;
import org.apache.fineract.portfolio.loanaccount.loanschedule.data.LoanSchedulePlanDownPaymentPeriod;
import org.apache.fineract.portfolio.loanaccount.loanschedule.data.LoanSchedulePlanPeriod;
import org.apache.fineract.portfolio.loanaccount.loanschedule.data.LoanSchedulePlanRepaymentPeriod;
import org.apache.fineract.portfolio.loanaccount.loanschedule.domain.EmbeddableProgressiveLoanScheduleGenerator;
import org.apache.fineract.portfolio.loanaccount.loanschedule.domain.LoanRepaymentScheduleModelData;

import java.math.BigDecimal;
import java.math.MathContext;
import java.math.RoundingMode;
import java.time.LocalDate;

public class Main {
  public static void main(String[] args) throws InterruptedException {
    MathContext mc = new MathContext(12, RoundingMode.HALF_UP);
    EmbeddableProgressiveLoanScheduleGenerator calculator = new EmbeddableProgressiveLoanScheduleGenerator();

    final CurrencyData currency = new CurrencyData("usd", "US Dollar", 2, null, "usd", "$");
    final LocalDate startDate = LocalDate.of(2024, 1, 1);
    final LocalDate disbursementDate = LocalDate.of(2024, 1, 1);
    final BigDecimal disbursedAmount = BigDecimal.valueOf(100);

    final int noRepayments = 6;
    final int repaymentFrequency = 1;
    final String repaymentFrequencyType = "MONTHS";
    final BigDecimal downPaymentPercentage = args.length > 0 ? new BigDecimal(args[0]) : BigDecimal.ZERO;
    final boolean isDownPaymentEnabled = BigDecimal.ZERO.compareTo(downPaymentPercentage) != 0;
    final BigDecimal annualNominalInterestRate = BigDecimal.valueOf(7.0);
    final DaysInMonthType daysInMonthType = DaysInMonthType.DAYS_30;
    final DaysInYearType daysInYearType = DaysInYearType.DAYS_360;
    final Integer installmentAmountInMultiplesOf = null;
    final Integer fixedLength = null;

    var config = new LoanRepaymentScheduleModelData(startDate, currency, disbursedAmount, disbursementDate, noRepayments, repaymentFrequency, repaymentFrequencyType, annualNominalInterestRate, isDownPaymentEnabled, daysInMonthType, daysInYearType, downPaymentPercentage, installmentAmountInMultiplesOf, fixedLength);

    final LoanSchedulePlan plan = calculator.generate(mc, config);
    printPlan(plan);
  }

  static void printPlan(final LoanSchedulePlan plan) throws InterruptedException {
    System.out.println("#------ Loan Schedule -----------------#");
    System.out.printf("  Number of Periods: %d%n", plan.getPeriods().stream().filter(period -> !(period instanceof LoanSchedulePlanDisbursementPeriod)).count());
    System.out.printf("  Loan Term in Days: %d%n", plan.getLoanTermInDays());
    System.out.printf("  Total Disbursed Amount: %s%n", plan.getTotalDisbursedAmount());
    System.out.printf("  Total Interest Amount: %s%n", plan.getTotalInterestAmount());
    System.out.printf("  Total Repayment Amount: %s%n", plan.getTotalRepaymentAmount());
    System.out.println("#------ Repayment Schedule ------------#");

    for (LoanSchedulePlanPeriod period : plan.getPeriods()) {
      if (period instanceof LoanSchedulePlanDisbursementPeriod dp) {
        System.out.printf("  Disbursement - Date: %s, Amount: %s%n", dp.periodDueDate(), dp.getPrincipalAmount());
      } if (period instanceof LoanSchedulePlanDownPaymentPeriod rp) {
        System.out.printf("  Down payment Period: #%d, Due Date: %s, Balance: %s, Principal: %s, Total: %s%n", rp.periodNumber(), rp.periodDueDate(), rp.getOutstandingLoanBalance(), rp.getPrincipalAmount(), rp.getTotalDueAmount());
      } if (period instanceof LoanSchedulePlanRepaymentPeriod rp) {
        System.out.printf("  Repayment Period: #%d, Due Date: %s, Balance: %s, Principal: %s, Interest: %s, Total: %s%n", rp.periodNumber(), rp.periodDueDate(), rp.getOutstandingLoanBalance(), rp.getPrincipalAmount(), rp.getInterestAmount(), rp.getTotalDueAmount());
      }
    }
  }
}
```


The project directory structure:
```
  src/
    Main.java
  libs/
    fineract-embeddable-calculator-1.11.0-SNAPSHOT-all.jar
  out/
    <<EMPTY>>
```

- Check Java minimum version

  ```shell
  java -version # openjdk version "17.0.3" 2022-04-19 LTS
  ```

- Compile the source

  ```shell
  javac -cp libs/fineract-progressive-loan-embeddable-schedule-generator-1.11.0-SNAPSHOT-all.jar -d out src/Main.java
  ```

- Run with dependencies:

  ```shell
  java -cp out:libs/fineract-progressive-loan-embeddable-schedule-generator-1.11.0-SNAPSHOT-all.jar Main
  ```

This code has the following output:

```
#------ Loan Schedule -----------------#
  Number of Periods: 6
  Loan Term in Days: 182
  Total Disbursed Amount: 100.00
  Total Interest Amount: 2.05
  Total Repayment Amount: 102.05
#------ Repayment Schedule ------------#
  Disbursement - Date: 2024-01-01, Amount: 100.00
  Repayment Period: #1, Due Date: 2024-02-01, Balance: 83.57, Principal: 16.43, Interest: 0.58, Total: 17.01
  Repayment Period: #2, Due Date: 2024-03-01, Balance: 67.05, Principal: 16.52, Interest: 0.49, Total: 17.01
  Repayment Period: #3, Due Date: 2024-04-01, Balance: 50.43, Principal: 16.62, Interest: 0.39, Total: 17.01
  Repayment Period: #4, Due Date: 2024-05-01, Balance: 33.71, Principal: 16.72, Interest: 0.29, Total: 17.01
  Repayment Period: #5, Due Date: 2024-06-01, Balance: 16.90, Principal: 16.81, Interest: 0.20, Total: 17.01
  Repayment Period: #6, Due Date: 2024-07-01, Balance: 0.00, Principal: 16.90, Interest: 0.10, Total: 17.00
```
