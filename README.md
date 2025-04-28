
import tkinter as tk
from tkinter import messagebox, ttk
import csv
import os

# File name to store records
FILE_NAME = "Record.csv"

def initialize_file():
    """Create the CSV file with headers if it doesn't exist."""
    if not os.path.exists(FILE_NAME):
        with open(FILE_NAME, mode="w", newline="") as file:
            writer = csv.writer(file)
            writer.writerow([
                "Check_In_Date", "Check_In_Time", "Room_No", "Name", "Address", "Phone_No",
                "ID_Proof", "Nationality", "Stay_Duration", "Payment_Mode", "Bed_Type",
                "Check_Out_Date", "Check_Out_Time"
            ])

initialize_file()

def save_record(data):
    """Save a new record to the CSV file."""
    with open(FILE_NAME, mode="a", newline="") as file:
        writer = csv.writer(file)
        writer.writerow(data)

def get_all_records():
    """Retrieve all records from the CSV file."""
    with open(FILE_NAME, mode="r") as file:
        reader = csv.reader(file)
        return list(reader)[1:]  # Skip header

def update_record(name, updated_data):
    """Update a record by name."""
    records = get_all_records()
    updated = False

    with open(FILE_NAME, mode="w", newline="") as file:
        writer = csv.writer(file)
        writer.writerow([
            "Check_In_Date", "Check_In_Time", "Room_No", "Name", "Address", "Phone_No",
            "ID_Proof", "Nationality", "Stay_Duration", "Payment_Mode", "Bed_Type",
            "Check_Out_Date", "Check_Out_Time"
        ])
        for record in records:
            if record[3] == name:  # Check if the name matches
                writer.writerow(updated_data)
                updated = True
            else:
                writer.writerow(record)

    return updated

def delete_record(name):
    """Delete a record by name."""
    records = get_all_records()
    deleted = False

    with open(FILE_NAME, mode="w", newline="") as file:
        writer = csv.writer(file)
        writer.writerow([
            "Check_In_Date", "Check_In_Time", "Room_No", "Name", "Address", "Phone_No",
            "ID_Proof", "Nationality", "Stay_Duration", "Payment_Mode", "Bed_Type",
            "Check_Out_Date", "Check_Out_Time"
        ])
        for record in records:
            if record[3] == name:  # Check if the name matches
                deleted = True
            else:
                writer.writerow(record)

    return deleted

# Tkinter GUI
class HotelManagementApp:
    def __init__(self, root):
        self.root = root
        self.root.title("Hotel Management System")
        self.root.geometry("800x600")

        # Add tabs
        self.notebook = ttk.Notebook(self.root)
        self.notebook.pack(expand=True, fill="both")

        self.add_room_booking_tab()
        self.add_view_records_tab()
        self.add_edit_update_tab()

    def add_room_booking_tab(self):
        self.booking_tab = tk.Frame(self.notebook)
        self.notebook.add(self.booking_tab, text="Room Booking")

        # Use grid for the title as well
        tk.Label(self.booking_tab, text="Room Booking", font=("Arial", 18)).grid(row=0, column=0, columnspan=2, pady=10)

        # Input Fields
        self.fields = {}
        labels = [
            "Check-In Date", "Check-In Time", "Room No", "Name", "Address", "Phone No",
            "ID Proof", "Nationality", "Stay Duration", "Payment Mode", "Bed Type",
            "Check-Out Date", "Check-Out Time"
        ]

        for i, label in enumerate(labels):
            tk.Label(self.booking_tab, text=label).grid(row=i + 1, column=0, padx=10, pady=5, sticky="w")
            entry = tk.Entry(self.booking_tab)
            entry.grid(row=i + 1, column=1, padx=10, pady=5)
            self.fields[label] = entry

        # Buttons
        tk.Button(self.booking_tab, text="Save", command=self.save_booking).grid(row=len(labels) + 1, column=0, pady=10, columnspan=2)

    def save_booking(self):
        data = [field.get() for field in self.fields.values()]

        # Validate mandatory fields
        if not all(data):
            messagebox.showerror("Error", "All fields must be filled!")
            return

        save_record(data)
        messagebox.showinfo("Success", "Booking saved successfully!")

        # Clear input fields
        for field in self.fields.values():
            field.delete(0, tk.END)

    def add_view_records_tab(self):
        self.view_tab = tk.Frame(self.notebook)
        self.notebook.add(self.view_tab, text="View Records")

        tk.Label(self.view_tab, text="Records", font=("Arial", 18)).pack(pady=10)

        # Table for viewing records
        self.tree = ttk.Treeview(self.view_tab, columns=(
            "Check-In Date", "Check-In Time", "Room No", "Name", "Address", "Phone No",
            "ID Proof", "Nationality", "Stay Duration", "Payment Mode", "Bed Type",
            "Check-Out Date", "Check-Out Time"
        ), show="headings")

        for col in self.tree["columns"]:
            self.tree.heading(col, text=col)
            self.tree.column(col, width=100)

        self.tree.pack(expand=True, fill="both", padx=10, pady=10)
        self.load_records()

        tk.Button(self.view_tab, text="Refresh", command=self.load_records).pack(pady=10)

    def load_records(self):
        for row in self.tree.get_children():
            self.tree.delete(row)

        records = get_all_records()
        for record in records:
            self.tree.insert("", tk.END, values=record)

    def add_edit_update_tab(self):
        self.edit_tab = tk.Frame(self.notebook)
        self.notebook.add(self.edit_tab, text="Edit/Update Record")

        tk.Label(self.edit_tab, text="Edit/Update Records", font=("Arial", 18)).pack(pady=10)

        tk.Label(self.edit_tab, text="Name to Edit:").pack(pady=5)
        self.name_to_edit = tk.Entry(self.edit_tab)
        self.name_to_edit.pack(pady=5)

        tk.Button(self.edit_tab, text="Edit", command=self.edit_record).pack(pady=5)

    def edit_record(self):
        name = self.name_to_edit.get()
        if not name:
            messagebox.showerror("Error", "Please enter a name to edit!")
            return

        records = get_all_records()
        for record in records:
            if record[3] == name:
                self.fill_edit_form(record)
                return

        messagebox.showerror("Error", "Record not found!")

    def fill_edit_form(self, record):
        edit_window = tk.Toplevel(self.root)
        edit_window.title("Edit Record")

        fields = {}
        labels = [
            "Check-In Date", "Check-In Time", "Room No", "Name", "Address", "Phone No",
            "ID Proof", "Nationality", "Stay Duration", "Payment Mode", "Bed Type",
            "Check-Out Date", "Check-Out Time"
        ]

        for i, label in enumerate(labels):
            tk.Label(edit_window, text=label).grid(row=i, column=0, padx=10, pady=5, sticky="w")
            entry = tk.Entry(edit_window)
            entry.insert(0, record[i])
            entry.grid(row=i, column=1, padx=10, pady=5)
            fields[label] = entry

        def save_changes():
            updated_data = [field.get() for field in fields.values()]
            update_record(record[3], updated_data)
            messagebox.showinfo("Success", "Record updated successfully!")
            edit_window.destroy()
            self.load_records()

        tk.Button(edit_window, text="Save Changes", command=save_changes).grid(row=len(labels), column=0, pady=10)

if __name__ == "__main__":
    root = tk.Tk()
    app = HotelManagementApp(root)
    root.mainloop()
