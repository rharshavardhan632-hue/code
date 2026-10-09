# code
code lost and found
CREATE DATABASE IF NOT EXISTS lostsystem;

USE lostsystem;

DROP TABLE IF EXISTS found_items;
DROP TABLE IF EXISTS lost_items;
DROP TABLE IF EXISTS users;


CREATE TABLE users (
    user_id INT PRIMARY KEY,
    name VARCHAR(50),
    class VARCHAR(20)
);


CREATE TABLE lost_items (
    lost_id INT PRIMARY KEY,
    user_id INT,
    item_name VARCHAR(50),
    category VARCHAR(30),
    color VARCHAR(50),
    location VARCHAR(100),
    lost_date DATE,
    description VARCHAR(255),
    status VARCHAR(20) DEFAULT 'Lost',

    FOREIGN KEY (user_id)
    REFERENCES users(user_id)
);


CREATE TABLE found_items (
    found_id INT PRIMARY KEY,
    user_id INT,
    item_name VARCHAR(50),
    category VARCHAR(30),
    color VARCHAR(50),
    location VARCHAR(100),
    found_date DATE,
    description VARCHAR(255),
    status VARCHAR(20) DEFAULT 'Found',

    FOREIGN KEY (user_id)
    REFERENCES users(user_id)
);











import mysql.connector as m


# ---------- DATABASE CONNECTION ----------

c = m.connect(
    host="localhost",
    user="root",
    passwd="root",
    database="lostsystem")

cr = c.cursor()


# ---------- CREATE / CHECK USER ----------

def get_or_create_user():

    user_id = int(input("Enter your User ID: "))

    cr.execute(
        "SELECT name, class FROM users WHERE user_id = %s",
        (user_id,)
    )

    user = cr.fetchone()

    # User already exists
    if user is not None:

        print("\nUser already exists!")
        print("Name:", user[0])
        print("Class:", user[1])

        return user_id

    # New user
    print("\nNew user detected. Creating user...")

    name = input("Enter your name: ")
    student_class = input("Enter your class: ")

    cr.execute(
        """
        INSERT INTO users
        (user_id, name, class)
        VALUES (%s, %s, %s)
        """,
        (user_id, name, student_class)
    )

    c.commit()

    print("User created successfully!")

    return user_id


# ---------- REPORT LOST ITEM ----------

def report_lost():

    print("\n========== REPORT LOST ITEM ==========")

    lost_id = int(input("Enter Lost Item ID: "))

    user_id = get_or_create_user()

    item = input("Item name: ")
    category = input("Category: ")
    color = input("Color: ")
    location = input("Location lost: ")
    date = input("Lost date (YYYY-MM-DD): ")
    description = input("Description: ")

    cr.execute(
        """
        INSERT INTO lost_items
        (lost_id, user_id, item_name, category,
         color, location, lost_date, description)
        VALUES (%s, %s, %s, %s, %s, %s, %s, %s)
        """,
        (
            lost_id,
            user_id,
            item,
            category,
            color,
            location,
            date,
            description
        )
    )

    c.commit()

    print("\nLost item registered successfully!")


# ---------- REPORT FOUND ITEM ----------

def report_found():

    print("\n========== REPORT FOUND ITEM ==========")

    found_id = int(input("Enter Found Item ID: "))

    user_id = get_or_create_user()

    item = input("Item name: ")
    category = input("Category: ")
    color = input("Color: ")
    location = input("Location found: ")
    date = input("Found date (YYYY-MM-DD): ")
    description = input("Description: ")

    cr.execute(
        """
        INSERT INTO found_items
        (found_id, user_id, item_name, category,
         color, location, found_date, description)
        VALUES (%s, %s, %s, %s, %s, %s, %s, %s)
        """,
        (
            found_id,
            user_id,
            item,
            category,
            color,
            location,
            date,
            description
        )
    )

    c.commit()

    print("\nFound item registered successfully!")


# ---------- VIEW LOST ITEMS ----------

def view_lost():

    cr.execute(
        """
        SELECT lost_id, user_id, item_name,
               category, color, location,
               lost_date, status
        FROM lost_items
        """
    )

    rows = cr.fetchall()

    print("\n========== LOST ITEMS ==========")





    if len(rows) == 0:

        print("No lost items.")

    else:

        for row in rows:
            print(row)


# ---------- VIEW FOUND ITEMS ----------

def view_found():

    cr.execute(
        """
        SELECT found_id, user_id, item_name,
               category, color, location,
               found_date, status
        FROM found_items
        """
    )

    rows = cr.fetchall()

    print("\n========== FOUND ITEMS ==========")

    if len(rows) == 0:

        print("No found items.")

    else:

        for row in rows:
            print(row)


# ---------- SEARCH FOUND ITEMS ----------

def search_items():

    keyword = input("Enter item name to search: ")

    cr.execute(
        """
        SELECT found_id, item_name, category,
               color, location, found_date
        FROM found_items
        WHERE item_name LIKE %s
        AND status = 'Found'
        """,
        ("%" + keyword + "%",)
    )

    rows = cr.fetchall()

    print("\n========== SEARCH RESULTS ==========")

    if len(rows) == 0:

        print("No matching items found.")

    else:

        for row in rows:
            print(row)


# ---------- FIND POSSIBLE MATCH ----------

def find_match():

    lost_id = int(input("Enter Lost Item ID: "))

    # Get lost item
    cr.execute(
        """
        SELECT item_name, category,
               color, location
        FROM lost_items
        WHERE lost_id = %s
        AND status = 'Lost'
        """,
        (lost_id,)
    )

    lost = cr.fetchone()

    if lost is None:

        print("\nLost item not found!")
        return


    # Convert lost information to lowercase
    lost_name = lost[0].lower()
    lost_category = lost[1].lower()
    lost_color = lost[2].lower()
    lost_location = lost[3].lower()


    # Get all active found items
    cr.execute(
        """
        SELECT found_id, item_name,
               category, color,
               location, found_date
        FROM found_items
        WHERE status = 'Found'
        """
    )

    found_items = cr.fetchall()

    matches = []


    # Check every found item
    for item in found_items:

        score = 0

        found_id = item[0]

        found_name = item[1].lower()
        found_category = item[2].lower()
        found_color = item[3].lower()
        found_location = item[4].lower()


        # ---------- ITEM NAME ----------

        lost_words = lost_name.split()
        found_words = found_name.split()

        for word in found_words:

            if word in lost_words:

                score += 40
                break


        # ---------- CATEGORY ----------

        if lost_category == found_category:

            score += 20

        elif lost_category in found_category:

            score += 10

        elif found_category in lost_category:

            score += 10


        # ---------- COLOR ----------

        lost_colors = lost_color.split()
        found_colors = found_color.split()

        for color in found_colors:

            if color in lost_colors:

                score += 15
                break


        # ---------- LOCATION ----------

        if lost_location == found_location:

            score += 15

        elif lost_location in found_location:

            score += 10

        elif found_location in lost_location:

            score += 10


        # ---------- SAVE MATCH ----------

        if score >= 50:

            matches.append(
                (found_id, score, item)
            )


    # ---------- DISPLAY MATCHES ----------

    print("\n========== POSSIBLE MATCHES ==========")

    if len(matches) == 0:

        print("No possible matches found.")

    else:

        matches.sort(
            key=lambda x: x[1],
            reverse=True
        )

        for match in matches:

            print("\nFound ID:", match[0])

            print(
                "Match Score:",
                str(match[1]) + "%"
            )

            print(
                "Details:",
                match[2]
            )


# ---------- MARK AS RETURNED ----------



def mark_returned():

    choice = input(
        "Lost item or Found item? (L/F): "
    ).upper()

    item_id = int(input("Enter ID: "))


    if choice == "L":

        cr.execute(
            """
            UPDATE lost_items
            SET status = 'Returned'
            WHERE lost_id = %s
            """,
            (item_id,)
        )


    elif choice == "F":

        cr.execute(
            """
            UPDATE found_items
            SET status = 'Returned'
            WHERE found_id = %s
            """,
            (item_id,)
        )


    else:

        print("Invalid choice!")
        return


    c.commit()

    print("Item status updated!")


# ---------- MAIN MENU ----------

while True:

    print("""
========================================
       LOST & FOUND MATCHING SYSTEM
========================================

1. Report Lost Item
2. Report Found Item
3. View Lost Items
4. View Found Items
5. Search Found Items
6. Find Possible Match
7. Mark Item as Returned
8. Exit
""")


    choice = input("Enter your choice: ")


    if choice == "1":

        report_lost()


    elif choice == "2":

        report_found()


    elif choice == "3":

        view_lost()


    elif choice == "4":

        view_found()


    elif choice == "5":

        search_items()


    elif choice == "6":

        find_match()


    elif choice == "7":

        mark_returned()


    elif choice == "8":

        print(
            "\nThank you for using Lost & Found System!"
        )

        break


    else:

        print("\nInvalid choice!")


c.close()
