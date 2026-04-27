# java
import java.util.Scanner;
import java.io.FileReader;
import java.io.PrintWriter;

public class LibrarySystem {

    // ---------- Book fields ----------
    static String[]  bookID    = new String[100];
    static String[]  title     = new String[100];
    static String[]  author    = new String[100];
    static boolean[] available = new boolean[100];
    static int bookCount = 0;

    // ---------- Loan fields ----------
    static String[] loanID   = new String[100];
    static String[] loanBook = new String[100];
    static String[] member   = new String[100];
    static String[] loanDate = new String[100];
    static int loanCount = 0;

    static Scanner input = new Scanner(System.in);

    // ============================================================
    public static void main(String[] args) throws Exception {

        loadBooks();
        loadLoans();

        int choice;

        do {
            System.out.println("\n====== Library Management System ======");
            System.out.println("1. Add a Book");
            System.out.println("2. Remove a Book");
            System.out.println("3. Display All Books");
            System.out.println("4. Checkout a Book");
            System.out.println("5. Return a Book");
            System.out.println("6. View All Loans");
            System.out.println("7. Exit");
            System.out.print("Enter choice: ");
            choice = Integer.parseInt(input.nextLine().trim());

            if      (choice == 1) addBook();
            else if (choice == 2) removeBook();
            else if (choice == 3) displayBooks();
            else if (choice == 4) checkoutBook();
            else if (choice == 5) returnBook();
            else if (choice == 6) viewLoans();
            else if (choice == 7) System.out.println("Goodbye!");
            else                  System.out.println("Invalid choice.");

        } while (choice != 7);
    }

    // ============================================================
    static void addBook() throws Exception {
        System.out.print("Enter Book ID: ");
        String id = input.nextLine().trim();

        for (int i = 0; i < bookCount; i++) {
            if (bookID[i].equalsIgnoreCase(id)) {
                System.out.println("That ID already exists.");
                return;
            }
        }

        System.out.print("Enter Title: ");
        String t = input.nextLine().trim();
        System.out.print("Enter Author: ");
        String a = input.nextLine().trim();

        bookID[bookCount]    = id;
        title[bookCount]     = t;
        author[bookCount]    = a;
        available[bookCount] = true;
        bookCount++;

        saveBooks();
        System.out.println("Book added!");
    }

    // ============================================================
    static void removeBook() throws Exception {
        System.out.print("Enter Book ID to remove: ");
        int i = findBook(input.nextLine().trim());

        if (i == -1) {
            System.out.println("Book not found.");
        } else if (!available[i]) {
            System.out.println("Cannot remove — book is checked out.");
        } else {
            for (int j = i; j < bookCount - 1; j++) {
                bookID[j]    = bookID[j+1];
                title[j]     = title[j+1];
                author[j]    = author[j+1];
                available[j] = available[j+1];
            }
            bookCount--;
            saveBooks();
            System.out.println("Book removed.");
        }
    }

    // ============================================================
    static void displayBooks() {
        if (bookCount == 0) { System.out.println("No books found."); return; }
        System.out.println("\n--- All Books ---");
        for (int i = 0; i < bookCount; i++) {
            String status = available[i] ? "Available" : "Checked Out";
            System.out.println((i+1) + ". ID: " + bookID[i] +
                " | " + title[i] + " by " + author[i] + " | " + status);
        }
    }

    // ============================================================
    static void checkoutBook() throws Exception {
        System.out.print("Enter Book ID to checkout: ");
        String id = input.nextLine().trim();
        int i = findBook(id);

        if (i == -1) {
            System.out.println("Book not found.");
        } else if (!available[i]) {
            System.out.println("Book is already checked out.");
        } else {
            System.out.print("Enter Member Name: ");
            String name = input.nextLine().trim();
            System.out.print("Enter Date (MM/DD/YYYY): ");
            String date = input.nextLine().trim();

            loanID[loanCount]   = "L" + (loanCount + 1);
            loanBook[loanCount] = id;
            member[loanCount]   = name;
            loanDate[loanCount] = date;
            loanCount++;
            available[i] = false;

            saveBooks();
            saveLoans();
            System.out.println("Checked out! Loan ID: L" + loanCount);
        }
    }

    // ============================================================
    static void returnBook() throws Exception {
        System.out.print("Enter Loan ID to return (e.g. L1): ");
        int i = findLoan(input.nextLine().trim());

        if (i == -1) {
            System.out.println("Loan not found.");
        } else {
            int b = findBook(loanBook[i]);
            if (b != -1) available[b] = true;

            for (int j = i; j < loanCount - 1; j++) {
                loanID[j]   = loanID[j+1];
                loanBook[j] = loanBook[j+1];
                member[j]   = member[j+1];
                loanDate[j] = loanDate[j+1];
            }
            loanCount--;
            saveBooks();
            saveLoans();
            System.out.println("Book returned successfully.");
        }
    }

    // ============================================================
    static void viewLoans() {
        if (loanCount == 0) { System.out.println("No active loans."); return; }
        System.out.println("\n--- Active Loans ---");
        for (int i = 0; i < loanCount; i++) {
            System.out.println((i+1) + ". " + loanID[i] +
                " | Book: " + loanBook[i] +
                " | Member: " + member[i] +
                " | Date: " + loanDate[i]);
        }
    }

    // ============================================================
    static int findBook(String id) {
        for (int i = 0; i < bookCount; i++)
            if (bookID[i].equalsIgnoreCase(id)) return i;
        return -1;
    }

    static int findLoan(String id) {
        for (int i = 0; i < loanCount; i++)
            if (loanID[i].equalsIgnoreCase(id)) return i;
        return -1;
    }

    // ============================================================
    static void saveBooks() throws Exception {
        PrintWriter out = new PrintWriter("books.txt");
        for (int i = 0; i < bookCount; i++)
            out.println(bookID[i] + "," + title[i] + "," + author[i] + "," + available[i]);
        out.close();
    }

    static void loadBooks() throws Exception {
        try {
            Scanner in = new Scanner(new FileReader("books.txt"));
            while (in.hasNextLine()) {
                String line = in.nextLine().trim();
                if (line.length() == 0) continue;
                int c1 = line.indexOf(",");
                int c2 = line.indexOf(",", c1 + 1);
                int c3 = line.indexOf(",", c2 + 1);
                bookID[bookCount]    = line.substring(0, c1);
                title[bookCount]     = line.substring(c1+1, c2);
                author[bookCount]    = line.substring(c2+1, c3);
                available[bookCount] = line.substring(c3+1).equals("true");
                bookCount++;
            }
            in.close();
        } catch (Exception e) {}
    }

    static void saveLoans() throws Exception {
        PrintWriter out = new PrintWriter("loans.txt");
        for (int i = 0; i < loanCount; i++)
            out.println(loanID[i] + "," + loanBook[i] + "," + member[i] + "," + loanDate[i]);
        out.close();
    }

    static void loadLoans() throws Exception {
        try {
            Scanner in = new Scanner(new FileReader("loans.txt"));
            while (in.hasNextLine()) {
                String line = in.nextLine().trim();
                if (line.length() == 0) continue;
                int c1 = line.indexOf(",");
                int c2 = line.indexOf(",", c1 + 1);
                int c3 = line.indexOf(",", c2 + 1);
                loanID[loanCount]   = line.substring(0, c1);
                loanBook[loanCount] = line.substring(c1+1, c2);
                member[loanCount]   = line.substring(c2+1, c3);
                loanDate[loanCount] = line.substring(c3+1);
                loanCount++;
            }
            in.close();
        } catch (Exception e) {}
    }
}
