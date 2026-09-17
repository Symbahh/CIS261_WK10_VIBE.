# Phillip Heard
# CIS261
# WK10 VIBE Coding
"""Student Grade Calculator: records, grades, search, statistics, and file I/O."""
from pathlib import Path
DATA_FILE = Path(__file__).with_name("student_grades.txt")
FIELDS = ("name", "id", "test1", "test2", "test3", "average", "grade")
def calculate_grade(average):
"""Return the letter grade for an average between 0 and 100."""
if average >= 90:
return "A"
if average >= 80:
return "B"
if average >= 70:
return "C"
if average >= 60:
return "D"
return "F"
def make_student(name, student_id, scores):
"""Build one dictionary and calculate its derived fields."""
average = sum(scores) / 3
return dict(zip(FIELDS, (name, student_id, *scores, average,
calculate_grade(average))))
def load_students(path=DATA_FILE):
"""Load valid pipe-delimited records; report invalid lines without crashing."""
students = []
try:
with path.open("r", encoding="utf-8") as source:
for line_number, line in enumerate(source, start=1):
parts = line.rstrip("\n\r").split("|")
try:
if len(parts) != 7 or not parts[0].strip() or not
parts[1].strip():
raise ValueError("missing fields")
scores = [float(value) for value in parts[2:5]]
if any(not 0 <= score <= 100 for score in scores):
raise ValueError("score outside 0-100")
if any(item["id"].casefold() == parts[1].strip().casefold()
for item in students):
raise ValueError("duplicate student ID")
# VIBE review: recalculate derived values instead of trusting
# stale average/grade values in a saved record.
students.append(make_student(parts[0].strip(),
parts[1].strip(), scores))
except ValueError as error:
print(f"Skipping invalid record on line {line_number}:
{error}.")
print(f"Loaded {len(students)} student(s) from {path.name}.")
except FileNotFoundError:
print("No saved student file found. Starting with an empty class.")
except OSError as error:
print(f"Could not load student records: {error}.")
return students
def save_students(students, path=DATA_FILE):
"""Save all records in the lab's required seven-field file format."""
try:
with path.open("w", encoding="utf-8") as destination:
for student in students:
destination.write("|".join((
student["name"], student["id"],
*(f'{student[f"test{number}"]:.2f}' for number in range(1, 4)),
f'{student["average"]:.2f}', student["grade"]
)) + "\n")
print(f"Saved {len(students)} student(s) to {path.name}.")
return True
except OSError as error:
print(f"Could not save student records: {error}.")
return False
def read_required(label):
while True:
value = input(label).strip()
if value.casefold() == "esc" or value == "\x1b":
return None
if value and "|" not in value and "\n" not in value:
return value
print("Enter a nonempty value without the | character.")
def read_score(number):
while True:
value = input(f"Test {number} score (0-100): ").strip()
if value.casefold() == "esc" or value == "\x1b":
return None
try:
score = float(value)
if 0 <= score <= 100:
return score
except ValueError:
pass
print("Enter a numeric score from 0 to 100, or ESC to cancel.")
def add_student(students):
name = read_required("Student name: ")
if name is None:
return
while True:
student_id = read_required("Student ID: ")
if student_id is None:
return
if not any(item["id"].casefold() == student_id.casefold()
for item in students):
break
print("That student ID already exists. Enter a different ID.")
scores = []
for number in range(1, 4):
score = read_score(number)
if score is None:
print("Student entry canceled.")
return
scores.append(score)
student = make_student(name, student_id, scores)
students.append(student)
print(f'Added {name}: average {student["average"]:.2f}, grade
{student["grade"]}.')
def display_students(students):
if not students:
print("No students to display.")
return
header = (f'{"Name":<24} {"ID":<12} {"Test 1":>7} {"Test 2":>7} '
f'{"Test 3":>7} {"Average":>7} {"Grade":>5}')
print(header)
print("-" * len(header))
for item in students:
print(f'{item["name"][:24]:<24} {item["id"][:12]:<12} '
f'{item["test1"]:>7.2f} {item["test2"]:>7.2f} '
f'{item["test3"]:>7.2f} {item["average"]:>7.2f} '
f'{item["grade"]:>5}')
def search_students(students):
query = read_required("Search name (whole or part): ")
if query is None:
return
matches = [item for item in students if query.casefold() in
item["name"].casefold()]
if matches:
print(f"Found {len(matches)} match(es):")
display_students(matches)
else:
print(f'No students found matching "{query}".')
def display_statistics(students):
if not students:
print("No students available for class statistics.")
return
highest = max(students, key=lambda item: item["average"])
lowest = min(students, key=lambda item: item["average"])
class_average = sum(item["average"] for item in students) / len(students)
print(f'Highest average: {highest["name"]} ({highest["average"]:.2f})')
print(f'Lowest average: {lowest["name"]} ({lowest["average"]:.2f})')
print(f"Class average: {class_average:.2f}")
def main():
# VIBE usage: AI-assisted first draft was reviewed and refined for score
# validation, duplicate IDs, reliable file loading, and ESC cancellation.
students = load_students()
while True:
print("\nStudent Grade Calculator")
print("1. Add student\n2. Display all students\n3. Search by name")
print("4. Class statistics\n5. Save and exit (or enter ESC)")
try:
choice = input("Choose an option: ").strip()
if choice == "1":
add_student(students)
elif choice == "2":
display_students(students)
elif choice == "3":
search_students(students)
elif choice == "4":
display_statistics(students)
elif choice == "5" or choice.casefold() == "esc" or choice == "\x1b":
if save_students(students):
print("Goodbye!")
break
print("Save failed. You can retry or exit with Ctrl+C.")
else:
print("Choose 1-5, or enter ESC to save and exit.")
except (EOFError, KeyboardInterrupt):
print("\nInput ended. Attempting to save before exiting.")
save_students(students)
break
if __name__ == "__main__":
main()
