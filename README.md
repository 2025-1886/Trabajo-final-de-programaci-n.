import sqlite3
from abc import ABC, abstractmethod


class BaseProduct(ABC):
    @abstractmethod
    def details(self):
        pass


class Product(BaseProduct):
    def __init__(self, name, brand, quantity):
        self.name = name
        self.brand = brand
        self.quantity = quantity

    def details(self):
        return f"{self.name} - {self.brand} - {self.quantity}"

    def add(self, quantity):
        self.quantity += quantity

    def update(self, new_quantity=None, add=None):
        if new_quantity is not None:
            self.quantity = new_quantity
        elif add is not None:
            self.quantity += add

    def delete(self):
        self.quantity = 0


class DatabaseConnection:
    def __init__(self):
        self.conn = sqlite3.connect("beauty.db")
        self.cursor = self.conn.cursor()
        self.create_table()

    def create_table(self):
        self.cursor.execute("""
        CREATE TABLE IF NOT EXISTS products (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            name TEXT,
            brand TEXT,
            quantity INTEGER
        )
        """)
        self.conn.commit()


class ProductRepository:
    def __init__(self):
        self.db = DatabaseConnection()

    def add(self, product):
        self.db.cursor.execute(
            "INSERT INTO products (name, brand, quantity) VALUES (?, ?, ?)",
            (product.name, product.brand, product.quantity)
        )
        self.db.conn.commit()

    def list_all(self):
        self.db.cursor.execute("SELECT * FROM products")
        return self.db.cursor.fetchall()

    def update(self, name, quantity):
        self.db.cursor.execute(
            "UPDATE products SET quantity=? WHERE name=?",
            (quantity, name)
        )
        self.db.conn.commit()

    def delete(self, name):
        self.db.cursor.execute(
            "DELETE FROM products WHERE name=?",
            (name,)
        )
        self.db.conn.commit()


class ProductService:
    def __init__(self):
        self.repo = ProductRepository()

    def register(self):
        name = input("Name: ")
        brand = input("Brand: ")

        try:
            quantity = int(input("Quantity: "))
        except:
            print("You must enter a number")
            return

        product = Product(name, brand, quantity)
        self.repo.add(product)
        print("Product saved")

    def list_products(self):
        products = self.repo.list_all()
        if not products:
            print("No products available")
        else:
            print("\n--- PRODUCT LIST ---")
            for p in products:
                print(f"{p[1]} | {p[2]} | Quantity: {p[3]}")

    def update_product(self):
        name = input("Product to update: ")

        try:
            quantity = int(input("New quantity: "))
        except:
            print("You must enter a number")
            return

        self.repo.update(name, quantity)
        print("Updated")

    def delete_product(self):
        name = input("Product to delete: ")
        self.repo.delete(name)
        print("Deleted")


def menu():
    print("\n=== BEAUTY SYSTEM ===")
    print("1. Add product")
    print("2. List products")
    print("3. Update product")
    print("4. Delete product")
    print("5. Exit")


def main():
    print("STARTING PROGRAM...")

    service = ProductService()

    while True:
        menu()
        option = input("Select an option: ")

        if option == "1":
            service.register()

        elif option == "2":
            service.list_products()

        elif option == "3":
            service.update_product()

        elif option == "4":
            service.delete_product()

        elif option == "5":
            print("Exiting...")
            break

        else:
            print("Invalid option")


if __name__ == "__main__":
    main()
