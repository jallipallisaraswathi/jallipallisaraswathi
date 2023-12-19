- 👋 Hi, I’m @jallipallisaraswathi
- 👀 I’m interested in ...
- 🌱 I’m currently learning ...
- 💞️ I’m looking to collaborate on ...
- 📫 How to reach me ...

<!---
jallipallisaraswathi/jallipallisaraswathi is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
import secrets
import string
import hashlib
import sqlite3

class PasswordManager:
    def __init__(self, db_name='passwords.db'):
        self.db_name = db_name
        self.connection = sqlite3.connect(db_name)
        self.cursor = self.connection.cursor()

    def __del__(self):
        self.connection.close()

    def generate_password(self, length=12):
        characters = string.ascii_letters + string.digits + string.punctuation
        password = ''.join(secrets.choice(characters) for _ in range(length))
        return password

    def check_password_strength(self, password):
        # Implement your criteria for password strength checking
        # For simplicity, just checking if the length is at least 8 characters
        return len(password) >= 8

    def create_password_table(self):
        self.cursor.execute('''CREATE TABLE IF NOT EXISTS passwords
                               (username TEXT PRIMARY KEY, salt TEXT, hashed_password TEXT)''')
        self.connection.commit()

    def hash_password(self, password, salt):
        hashed_password = hashlib.sha256((password + salt).encode()).hexdigest()
        return hashed_password

    def store_password(self, username, password):
        salt = secrets.token_hex(8)  # Generate a random salt
        hashed_password = self.hash_password(password, salt)
        self.cursor.execute("INSERT OR REPLACE INTO passwords VALUES (?, ?, ?)",
                            (username, salt, hashed_password))
        self.connection.commit()

    def verify_password(self, username, password):
        self.cursor.execute("SELECT salt, hashed_password FROM passwords WHERE username=?", (username,))
        result = self.cursor.fetchone()
        if result:
            salt, stored_password = result
            hashed_password = self.hash_password(password, salt)
            return hashed_password == stored_password
        return False

# Example usage
manager = PasswordManager()

password = manager.generate_password()
print(f"Generated Password: {password}")

if manager.check_password_strength(password):
    manager.create_password_table()
    manager.store_password("example_user", password)

    # Verify the password
    if manager.verify_password("example_user", password):
        print("Password verified successfully!")
    else:
        print("Password verification failed.")
else:
    print("Generated password does not meet the criteria.")
