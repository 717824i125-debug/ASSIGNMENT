import java.util.*;
import java.time.LocalDate;

// ------------------ Customer ------------------
class Customer {
    private String id;
    private String name;

    public Customer(String id, String name) {
        this.id = id;
        this.name = name;
    }

    public String getId() { return id; }
    public String getName() { return name; }
}

// ------------------ LoanProduct ------------------
class LoanProduct {
    private String id;
    private String name;
    private double annualInterestRate;
    private int termMonths;

    public LoanProduct(String id, String name, double annualInterestRate, int termMonths) {
        this.id = id;
        this.name = name;
        this.annualInterestRate = annualInterestRate;
        this.termMonths = termMonths;
    }

    public String getId() { return id; }
    public String getName() { return name; }
    public double getAnnualInterestRate() { return annualInterestRate; }
    public int getTermMonths() { return termMonths; }
}

// ------------------ Loan ------------------
class Loan {
    private String id;
    private Customer customer;
    private LoanProduct product;
    private double amount;
    private boolean approved;
    private boolean disbursed;
    private List<RepaymentSchedule> schedules = new ArrayList<>();
    private double outstanding;

    public Loan(String id, Customer customer, LoanProduct product, double amount) {
        this.id = id;
        this.customer = customer;
        this.product = product;
        this.amount = amount;
        this.outstanding = amount;
    }

    public String getId() { return id; }
    public Customer getCustomer() { return customer; }
    public LoanProduct getProduct() { return product; }
    public double getAmount() { return amount; }
    public boolean isApproved() { return approved; }
    public boolean isDisbursed() { return disbursed; }
    public double getOutstanding() { return outstanding; }
    public List<RepaymentSchedule> getSchedules() { return schedules; }

    public void approve() { this.approved = true; }

    public void disburse() {
        this.disbursed = true;
        generateSchedule();
    }

    private void generateSchedule() {
        double principalPerMonth = amount / product.getTermMonths();
        double monthlyRate = product.getAnnualInterestRate() / 12 / 100;

        double remaining = amount;
        for (int i = 1; i <= product.getTermMonths(); i++) {
            double interest = remaining * monthlyRate;
            double principal = principalPerMonth;
            schedules.add(new RepaymentSchedule(i, principal, interest));
            remaining -= principal;
        }
    }

    public void recordRepayment(int installmentNo, double amountPaid) {
        if (installmentNo <= 0 || installmentNo > schedules.size()) {
            System.out.println("Invalid installment number.");
            return;
        }
        RepaymentSchedule sch = schedules.get(installmentNo - 1);
        if (sch.isPaid()) {
            System.out.println("This installment is already paid.");
            return;
        }
        double due = sch.getPrincipal() + sch.getInterest();
        double penalty = 0;

        if (amountPaid < due) {
            penalty = (due - amountPaid) * 0.02; // 2% penalty on shortfall
        }

        sch.markPaid(amountPaid, penalty);
        outstanding -= sch.getPrincipal();

        System.out.println("Repayment recorded successfully!");
        System.out.printf("Receipt -> Loan ID: %s | Installment: %d%n", id, installmentNo);
        System.out.printf("Principal Paid: %.2f | Interest Paid: %.2f | Penalty: %.2f%n",
                sch.getPrincipal(), sch.getInterest(), penalty);
        System.out.printf("Remaining Balance: %.2f%n", outstanding);
    }

    public String getStatus() {
        if (outstanding <= 0) return "Closed";
        for (RepaymentSchedule rs : schedules) {
            if (!rs.isPaid() && rs.getDueDate().isBefore(LocalDate.now())) {
                return "In Arrears";
            }
        }
        return "Active";
    }
}

// ------------------ RepaymentSchedule ------------------
class RepaymentSchedule {
    private int installmentNo;
    private double principal;
    private double interest;
    private boolean paid;
    private double amountPaid;
    private double penalty;
    private LocalDate dueDate;

    public RepaymentSchedule(int installmentNo, double principal, double interest) {
        this.installmentNo = installmentNo;
        this.principal = principal;
        this.interest = interest;
        this.paid = false;
        this.dueDate = LocalDate.now().plusMonths(installmentNo);
    }

    public int getInstallmentNo() { return installmentNo; }
    public double getPrincipal() { return principal; }
    public double getInterest() { return interest; }
    public boolean isPaid() { return paid; }
    public double getAmountPaid() { return amountPaid; }
    public double getPenalty() { return penalty; }
    public LocalDate getDueDate() { return dueDate; }

    public void markPaid(double amount, double penalty) {
        this.paid = true;
        this.amountPaid = amount;
        this.penalty = penalty;
    }
}

// ------------------ Main Application ------------------
public class MicrofinanceApp {
    private static Scanner sc = new Scanner(System.in);
    private static List<Customer> customers = new ArrayList<>();
    private static List<LoanProduct> products = new ArrayList<>();
    private static List<Loan> loans = new ArrayList<>();
    private static int loanCounter = 1;

    public static void main(String[] args) {
        while (true) {
            System.out.println("\n=== Microfinance Loans & Repayments System ===");
            System.out.println("1. Add Customer");
            System.out.println("2. Add Loan Product");
            System.out.println("3. Approve Loan");
            System.out.println("4. Disburse Loan");
            System.out.println("5. Generate Schedule");
            System.out.println("6. Record Repayment");
            System.out.println("7. View Outstanding");
            System.out.println("8. Exit");
            System.out.print("Choose: ");
            int choice = Integer.parseInt(sc.nextLine());

            switch (choice) {
                case 1 -> addCustomer();
                case 2 -> addProduct();
                case 3 -> approveLoan();
                case 4 -> disburseLoan();
                case 5 -> generateSchedule();
                case 6 -> recordRepayment();
                case 7 -> viewOutstanding();
                case 8 -> { System.out.println("Exiting system. Goodbye!"); return; }
                default -> System.out.println("Invalid choice!");
            }
        }
    }

    private static void addCustomer() {
        System.out.print("Enter Customer ID: ");
        String id = sc.nextLine();
        System.out.print("Enter Customer Name: ");
        String name = sc.nextLine();
        customers.add(new Customer(id, name));
        System.out.println("Customer added successfully!");
    }

    private static void addProduct() {
        System.out.print("Enter Loan Product ID: ");
        String id = sc.nextLine();
        System.out.print("Enter Product Name: ");
        String name = sc.nextLine();
        System.out.print("Enter Interest Rate (% per annum): ");
        double rate = Double.parseDouble(sc.nextLine());
        System.out.print("Enter Term (months): ");
        int term = Integer.parseInt(sc.nextLine());
        products.add(new LoanProduct(id, name, rate, term));
        System.out.println("Loan product added successfully!");
    }

    private static void approveLoan() {
        System.out.print("Enter Customer ID: ");
        String cid = sc.nextLine();
        Customer c = customers.stream().filter(x -> x.getId().equals(cid)).findFirst().orElse(null);
        if (c == null) { System.out.println("Customer not found!"); return; }

        System.out.print("Enter Loan Product ID: ");
        String pid = sc.nextLine();
        LoanProduct p = products.stream().filter(x -> x.getId().equals(pid)).findFirst().orElse(null);
        if (p == null) { System.out.println("Product not found!"); return; }

        System.out.print("Enter Loan Amount: ");
        double amt = Double.parseDouble(sc.nextLine());
        String loanId = "LN" + String.format("%03d", loanCounter++);
        Loan loan = new Loan(loanId, c, p, amt);
        loan.approve();
        loans.add(loan);
        System.out.println("Loan approved successfully. Loan ID: " + loanId);
    }

    private static void disburseLoan() {
        System.out.print("Enter Loan ID: ");
        String lid = sc.nextLine();
        Loan loan = loans.stream().filter(x -> x.getId().equals(lid)).findFirst().orElse(null);
        if (loan == null) { System.out.println("Loan not found!"); return; }
        if (!loan.isApproved()) { System.out.println("Loan not approved!"); return; }
        if (loan.isDisbursed()) { System.out.println("Loan already disbursed!"); return; }
        loan.disburse();
        System.out.println("Loan " + lid + " disbursed successfully.");
    }

    private static void generateSchedule() {
        System.out.print("Enter Loan ID: ");
        String lid = sc.nextLine();
        Loan loan = loans.stream().filter(x -> x.getId().equals(lid)).findFirst().orElse(null);
        if (loan == null) { System.out.println("Loan not found!"); return; }
        if (!loan.isDisbursed()) { System.out.println("Loan not disbursed yet!"); return; }

        System.out.println("Repayment Schedule for Loan " + lid + ":");
        for (RepaymentSchedule rs : loan.getSchedules()) {
            System.out.printf("Installment %d -> Principal: %.2f, Interest: %.2f, Total: %.2f, Due: %s%n",
                    rs.getInstallmentNo(), rs.getPrincipal(), rs.getInterest(),
                    rs.getPrincipal() + rs.getInterest(), rs.getDueDate());
        }
        System.out.printf("Outstanding Loan Balance: %.2f%n", loan.getOutstanding());
    }

    private static void recordRepayment() {
        System.out.print("Enter Loan ID: ");
        String lid = sc.nextLine();
        Loan loan = loans.stream().filter(x -> x.getId().equals(lid)).findFirst().orElse(null);
        if (loan == null) { System.out.println("Loan not found!"); return; }
        if (!loan.isDisbursed()) { System.out.println("Loan not disbursed yet!"); return; }

        System.out.print("Enter Installment No: ");
        int inst = Integer.parseInt(sc.nextLine());
        System.out.print("Enter Payment Amount: ");
        double amt = Double.parseDouble(sc.nextLine());

        loan.recordRepayment(inst, amt);
    }

    private static void viewOutstanding() {
        System.out.print("Enter Loan ID: ");
        String lid = sc.nextLine();
        Loan loan = loans.stream().filter(x -> x.getId().equals(lid)).findFirst().orElse(null);
        if (loan == null) { System.out.println("Loan not found!"); return; }

        System.out.printf("Loan %s Status: %s%n", lid, loan.getStatus());
        System.out.printf("Outstanding Principal: %.2f%n", loan.getOutstanding());
    }
}
