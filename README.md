import java.text.DecimalFormat;
import java.util.ArrayList;
import java.util.List;
import java.util.Scanner;


public class InvoiceApp {
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);

        System.out.println("===== Invoice Generation and Tax Calculation System =====");

                    // Customer details
        System.out.print("Enter Customer Name: ");
        String cname = sc.nextLine();
        System.out.print("Enter Customer Phone: ");
        String cphone = sc.nextLine();
        Customer customer = new Customer(cname, cphone);

                    // Items
        List<Item> items = new ArrayList<>();
        while (true) {
            System.out.print("Add item? (yes/no): ");
            String ans = sc.nextLine().trim().toLowerCase();
            if (!ans.equals("yes")) break;

            System.out.print("Item Name: ");
            String iname = sc.nextLine();
            System.out.print("Price: ");
            double price = sc.nextDouble();
            System.out.print("Quantity: ");
            int qty = sc.nextInt();
            sc.nextLine(); 

            items.add(new Item(iname, price, qty));
        }

        System.out.println("Select Tax Rate (%): 0, 5, 12, 18, 28");
        double rate = sc.nextDouble();
        sc.close();
        TaxCalculator taxCalc = new GSTTax(rate / 100.0);

        Invoice invoice = new Invoice(customer, items, taxCalc);
        System.out.println("\n" + invoice.formatInvoice());
    }
}

class Customer {
    private String name;
    private String phone;

    public Customer(String name, String phone) {
        this.name = name;
        this.phone = phone;
    }

    public String getName() { return name; }
    public String getPhone() { return phone; }
}

class Item {
    private String name;
    private double price;
    private int quantity;

    public Item(String name, double price, int quantity) {
        this.name = name;
        this.price = price;
        this.quantity = quantity;
    }

    public String getName() { return name; }
    public double getPrice() { return price; }
    public int getQuantity() { return quantity; }
    public double getLineTotal() { return price * quantity; }
}

interface TaxCalculator {
    double calculateTax(double amount);
}

class GSTTax implements TaxCalculator {
    private final double rate;

    public GSTTax(double rate) { this.rate = rate; }

    @Override
    public double calculateTax(double amount) {
        return amount * rate;
    }
}

class Invoice {
    private Customer customer;
    private List<Item> items;
    private TaxCalculator taxCalculator;
    private final DecimalFormat df = new DecimalFormat("0.00");

    public Invoice(Customer customer, List<Item> items, TaxCalculator taxCalculator) {
        this.customer = customer;
        this.items = new ArrayList<>(items);
        this.taxCalculator = taxCalculator;
    }

    public double getSubtotal() {
        double s = 0;
        for (Item it : items) s += it.getLineTotal();
        return s;
    }

    public double getTax() {
        return taxCalculator.calculateTax(getSubtotal());
    }

    public double getTotal() {
        return getSubtotal() + getTax();
    }

    public String formatInvoice() {
        StringBuilder sb = new StringBuilder();
        sb.append("INVOICE\n");
        sb.append("----------------------------------------\n");
        sb.append("Customer: ").append(customer.getName()).append("\n");
        if (customer.getPhone() != null && !customer.getPhone().isEmpty())
            sb.append("Phone: ").append(customer.getPhone()).append("\n");
        sb.append("----------------------------------------\n");
        sb.append(String.format("%-20s %8s %6s %12s\n", "Item", "Price", "Qty", "Line Total"));
        sb.append("----------------------------------------\n");
        for (Item it : items) {
            sb.append(String.format("%-20s %8s %6d %12s\n",
                    it.getName(), df.format(it.getPrice()), it.getQuantity(), df.format(it.getLineTotal())));
        }
        sb.append("----------------------------------------\n");
        sb.append(String.format("%-30s %10s\n", "Subtotal:", df.format(getSubtotal())));
        sb.append(String.format("%-30s %10s\n", "Tax:", df.format(getTax())));
        sb.append(String.format("%-30s %10s\n", "Total:", df.format(getTotal())));
        sb.append("----------------------------------------\n");
        sb.append("Thank you for your business!\n");
        return sb.toString();
     }
}
