# python-11
import json
import csv
import datetime
import os


def log_action(action):
    with open("log.txt", "a", encoding="utf-8") as f:
        f.write(f"{datetime.datetime.now()}: {action}\n")


class Contact:
    def __init__(self, name, phone, email, address):
        if not name or not phone:
            raise ValueError("Ism va telefon raqami majburiy!")
        if not phone.isdigit() or len(phone) < 7:
            raise ValueError("Noto'g'ri telefon raqami formati!")

        self.name = name
        self.phone = phone
        self.email = email
        self.address = address

    def to_dict(self):
        return {"name": self.name, "phone": self.phone, "email": self.email, "address": self.address}


class ContactBook:
    def __init__(self, filename="contacts.json"):
        self.filename = filename
        self.contacts = self.load_contacts()

    def load_contacts(self):
        try:
            if not os.path.exists(self.filename): return []
            with open(self.filename, "r", encoding="utf-8") as f:
                return json.load(f)
        except (json.JSONDecodeError, Exception) as e:
            log_action(f"Xatolik yuklashda: {e}")
            return []

    def save(self):
        with open(self.filename, "w", encoding="utf-8") as f:
            json.dump(self.contacts, f, indent=4)
        log_action("Ma'lumotlar faylga saqlandi.")

    def add(self, contact):
        self.contacts.append(contact.to_dict())
        self.save()
        log_action(f"Kontakt qo'shildi: {contact.name}")

    def list_all(self):
        return self.contacts

    def delete(self, name):
        self.contacts = [c for c in self.contacts if c['name'] != name]
        self.save()
        log_action(f"Kontakt o'chirildi: {name}")

    def export_csv(self):
        with open("contacts.csv", "w", newline="", encoding="utf-8") as f:
            writer = csv.DictWriter(f, fieldnames=["name", "phone", "email", "address"])
            writer.writeheader()
            writer.writerows(self.contacts)
        log_action("CSV ga eksport qilindi.")


def main():
    book = ContactBook()
    while True:
        print("\n--- KONTAKT KITOBI ---")
        print("1. Qo'shish | 2. Ko'rish | 3. O'chirish | 4. CSV Eksport | 5. Chiqish")
        choice = input("Tanlang: ")

        try:
            if choice == '1':
                name = input("Ism: ")
                phone = input("Tel: ")
                c = Contact(name, phone, input("Email: "), input("Manzil: "))
                book.add(c)
                print("Muvaffaqiyatli qo'shildi!")
            elif choice == '2':
                for c in book.list_all(): print(c)
            elif choice == '3':
                book.delete(input("O'chirish uchun ism: "))
            elif choice == '4':
                book.export_csv()
            elif choice == '5':
                break
        except Exception as e:
            print(f"Xatolik: {e}")


if __name__ == "__main__":
    main()
