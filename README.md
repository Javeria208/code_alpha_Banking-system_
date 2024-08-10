#include <iostream>
#include <string>

// Constants for maximum sizes
const int MAX_CUSTOMERS = 100;
const int MAX_ACCOUNTS = 100;
const int MAX_TRANSACTIONS = 10; // Number of recent transactions to keep

// Transaction Class
class Transaction {
public:
    Transaction(int id = 0, double amt = 0.0, std::string date = "", std::string type = "")
        : transactionID(id), amount(amt), date(date), type(type) {}

    void getTransactionDetails() const {
        std::cout << "Transaction ID: " << transactionID
                  << "\nAmount: " << amount
                  << "\nDate: " << date
                  << "\nType: " << type << std::endl;
    }

private:
    int transactionID;
    double amount;
    std::string date;
    std::string type;
};

// Account Class
class Account {
public:
    Account(int num = 0, double bal = 0.0) 
        : accountNumber(num), balance(bal), numTransactions(0) {}

    void deposit(double amount, int transactionID, const std::string& date) {
        balance += amount;
        addTransaction(transactionID, amount, date, "Deposit");
    }

    void withdraw(double amount, int transactionID, const std::string& date) {
        if (amount <= balance) {
            balance -= amount;
            addTransaction(transactionID, amount, date, "Withdrawal");
        } else {
            std::cout << "Insufficient funds" << std::endl;
        }
    }

    void transfer(Account& toAccount, double amount, int transactionID, const std::string& date) {
        if (amount <= balance) {
            balance -= amount;
            toAccount.deposit(amount, transactionID, date);
            addTransaction(transactionID, amount, date, "Transfer to Account " + std::to_string(toAccount.getAccountNumber()));
        } else {
            std::cout << "Insufficient funds" << std::endl;
        }
    }

    double getBalance() const {
        return balance;
    }

    void getAccountDetails() const {
        std::cout << "Account Number: " << accountNumber
                  << "\nBalance: " << balance << std::endl;
        std::cout << "Recent Transactions:" << std::endl;
        for (int i = 0; i < numTransactions; ++i) {
            transactions[i].getTransactionDetails();
            std::cout << std::endl;
        }
    }

    int getAccountNumber() const {
        return accountNumber;
    }

private:
    void addTransaction(int transactionID, double amount, const std::string& date, const std::string& type) {
        if (numTransactions < MAX_TRANSACTIONS) {
            transactions[numTransactions++] = Transaction(transactionID, amount, date, type);
        } else {
            // Shift transactions to make room for the new one (FIFO)
            for (int i = 1; i < MAX_TRANSACTIONS; ++i) {
                transactions[i - 1] = transactions[i];
            }
            transactions[MAX_TRANSACTIONS - 1] = Transaction(transactionID, amount, date, type);
        }
    }

    int accountNumber;
    double balance;
    Transaction transactions[MAX_TRANSACTIONS];
    int numTransactions;
};

// Customer Class
class Customer {
public:
    Customer(int id = 0, const std::string& name = "") 
        : customerID(id), name(name), numAccounts(0) {}

    void createAccount(int accountNumber, double initialBalance) {
        if (numAccounts < MAX_ACCOUNTS) {
            accounts[numAccounts++] = Account(accountNumber, initialBalance);
        } else {
            std::cout << "Account limit reached" << std::endl;
        }
    }

    void viewAccountDetails(int accountNumber) const {
        for (int i = 0; i < numAccounts; ++i) {
            if (accounts[i].getAccountNumber() == accountNumber) {
                accounts[i].getAccountDetails();
                return;
            }
        }
        std::cout << "Account not found" << std::endl;
    }

    int getCustomerID() const {
        return customerID;
    }

    Account* getAccounts() {
        return accounts;
    }

    int getNumAccounts() const {
        return numAccounts;
    }

private:
    int customerID;
    std::string name;
    Account accounts[MAX_ACCOUNTS];
    int numAccounts;
};

// BankingServices Class
class BankingServices {
public:
    BankingServices() : numCustomers(0) {}

    void addCustomer(const Customer& customer) {
        if (numCustomers < MAX_CUSTOMERS) {
            customers[numCustomers++] = customer;
        } else {
            std::cout << "Customer limit reached" << std::endl;
        }
    }

    Customer* findCustomer(int customerID) {
        for (int i = 0; i < numCustomers; ++i) {
            if (customers[i].getCustomerID() == customerID) {
                return &customers[i];
            }
        }
        return nullptr;
    }

private:
    Customer customers[MAX_CUSTOMERS];
    int numCustomers;
};

int main() {
    BankingServices bank;
    int choice;

    while (true) {
        std::cout << "Banking System Menu:\n";
        std::cout << "1. Add Customer\n";
        std::cout << "2. Create Account\n";
        std::cout << "3. View Account Details\n";
        std::cout << "4. Deposit\n";
        std::cout << "5. Withdraw\n";
        std::cout << "6. Transfer\n";
        std::cout << "7. Exit\n";
        std::cout << "Enter your choice: ";
        std::cin >> choice;

        switch (choice) {
            case 1: {
                int id;
                std::string name;
                std::cout << "Enter customer ID: ";
                std::cin >> id;
                std::cout << "Enter customer name: ";
                std::cin.ignore(); // to ignore the newline character left in the buffer
                std::getline(std::cin, name);
                Customer newCustomer(id, name);
                bank.addCustomer(newCustomer);
                break;
            }
            case 2: {
                int customerID, accountNumber;
                double initialBalance;
                std::cout << "Enter customer ID: ";
                std::cin >> customerID;
                Customer* customer = bank.findCustomer(customerID);
                if (customer) {
                    std::cout << "Enter account number: ";
                    std::cin >> accountNumber;
                    std::cout << "Enter initial balance: ";
                    std::cin >> initialBalance;
                    customer->createAccount(accountNumber, initialBalance);
                } else {
                    std::cout << "Customer not found" << std::endl;
                }
                break;
            }
            case 3: {
                int customerID, accountNumber;
                std::cout << "Enter customer ID: ";
                std::cin >> customerID;
                Customer* customer = bank.findCustomer(customerID);
                if (customer) {
                    std::cout << "Enter account number: ";
                    std::cin >> accountNumber;
                    customer->viewAccountDetails(accountNumber);
                } else {
                    std::cout << "Customer not found" << std::endl;
                }
                break;
            }
            case 4: {
                int customerID, accountNumber, transactionID;
                double amount;
                std::string date;
                std::cout << "Enter customer ID: ";
                std::cin >> customerID;
                Customer* customer = bank.findCustomer(customerID);
                if (customer) {
                    std::cout << "Enter account number: ";
                    std::cin >> accountNumber;
                    Account* account = nullptr;
                    for (int i = 0; i < customer->getNumAccounts(); ++i) {
                        if (customer->getAccounts()[i].getAccountNumber() == accountNumber) {
                            account = &customer->getAccounts()[i];
                            break;
                        }
                    }
                    if (account) {
                        std::cout << "Enter transaction ID: ";
                        std::cin >> transactionID;
                        std::cout << "Enter deposit amount: ";
                        std::cin >> amount;
                        std::cout << "Enter transaction date (YYYY-MM-DD): ";
                        std::cin.ignore();
                        std::getline(std::cin, date);
                        account->deposit(amount, transactionID, date);
                    } else {
                        std::cout << "Account not found" << std::endl;
                    }
                } else {
                    std::cout << "Customer not found" << std::endl;
                }
                break;
            }
            case 5: {
                int customerID, accountNumber, transactionID;
                double amount;
                std::string date;
                std::cout << "Enter customer ID: ";
                std::cin >> customerID;
                Customer* customer = bank.findCustomer(customerID);
                if (customer) {
                    std::cout << "Enter account number: ";
                    std::cin >> accountNumber;
                    Account* account = nullptr;
                    for (int i = 0; i < customer->getNumAccounts(); ++i) {
                        if (customer->getAccounts()[i].getAccountNumber() == accountNumber) {
                            account = &customer->getAccounts()[i];
                            break;
                        }
                    }
                    if (account) {
                        std::cout << "Enter transaction ID: ";
                        std::cin >> transactionID;
                        std::cout << "Enter withdrawal amount: ";
                        std::cin >> amount;
                        std::cout << "Enter transaction date (YYYY-MM-DD): ";
                        std::cin.ignore();
                        std::getline(std::cin, date);
                        account->withdraw(amount, transactionID, date);
                    } else {
                        std::cout << "Account not found" << std::endl;
                    }
                } else {
                    std::cout << "Customer not found" << std::endl;
                }
                break;
            }
            case 6: {
                int customerID, fromAccountNumber, toAccountNumber, transactionID;
                double amount;
                std::string date;
                std::cout << "Enter customer ID: ";
                std::cin >> customerID;
                Customer* customer = bank.findCustomer(customerID);
                if (customer) {
                    std::cout << "Enter source account number: ";
                    std::cin >> fromAccountNumber;
                    Account* fromAccount = nullptr;
                    for (int i = 0; i < customer->getNumAccounts(); ++i) {
                        if (customer->getAccounts()[i].getAccountNumber() == fromAccountNumber) {
                            fromAccount = &customer->getAccounts()[i];
                            break;
                        }
                    }
                    if (fromAccount) {
                        std::cout << "Enter destination account number: ";
                        std::cin >> toAccountNumber;
                        Account* toAccount = nullptr;
                        for (int i = 0; i < customer->getNumAccounts(); ++i) {
                            if (customer->getAccounts()[i].getAccountNumber() == toAccountNumber) {
                                toAccount = &customer->getAccounts()[i];
                                break;
                            }
                        }
                        if (toAccount) {
                            std::cout << "Enter transaction ID: ";
                            std::cin >> transactionID;
                            std::cout << "Enter transfer amount: ";
                            std::cin >> amount;
                            std::cout << "Enter transaction date (YYYY-MM-DD): ";
                            std::cin.ignore();
                            std::getline(std::cin, date);
                            fromAccount->transfer(*toAccount, amount, transactionID, date);
                        } else {
                            std::cout << "Destination account not found" << std::endl;
                        }
                    } else {
                        std::cout << "Source account not found" << std::endl;
                    }
                } else {
                    std::cout << "Customer not found" << std::endl;
                }
                break;
            }
            case 7:
                return 0;
            default:
                std::cout << "Invalid choice. Please try again." << std::endl;
        }
    }
}
