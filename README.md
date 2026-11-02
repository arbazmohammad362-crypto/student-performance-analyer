# student-performance-analyer
project was build by using concept pandas,Numpy,matplotlib,statistics 
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import os

# Subject names
subjects = ["Maths", "Physics", "Chemistry"]


# Calculate grade
def calculate_grade(percentage):

    if percentage >= 90:
        return "A"
    elif percentage >= 75:
        return "B"
    elif percentage >= 60:
        return "C"
    elif percentage >= 40:
        return "D"
    else:
        return "F"


# Load CSV file and validate marks
def load_data():

    try:
        file_path = os.path.join(
            os.path.dirname(os.path.abspath(__file__)),
            "students.csv"
        )

        df = pd.read_csv(file_path, dtype=str)

    except FileNotFoundError:
        print("Error: students.csv file not found.")
        return pd.DataFrame()

    required_columns = ["Name", "Maths", "Physics", "Chemistry"]

    if not all(col in df.columns for col in required_columns):
        print("Error: Invalid CSV columns.")
        return pd.DataFrame()

    valid_rows = []

    for _, row in df.iterrows():

        name = row["Name"]

        if pd.isna(name) or not str(name).strip():
            print("Invalid student name. Row skipped.")
            continue

        marks = []
        invalid = False

        for subject in subjects:

            value = row[subject]
            mark = pd.to_numeric(value, errors="coerce")

            if pd.isna(mark) or mark < 0 or mark > 100:

                print(
                    f"Invalid marks for {name} in {subject}: "
                    f"'{value}'. Row skipped."
                )

                invalid = True
                break

            marks.append(float(mark))

        if not invalid:
            valid_rows.append([name] + marks)

    return pd.DataFrame(
        valid_rows,
        columns=required_columns
    )


# Calculate total, percentage and grade
def calculate_results(df):

    if df.empty:
        print("No valid student records available.")
        return df

    df = df.copy()

    df["Total"] = df[subjects].sum(axis=1)

    df["Percentage"] = (df["Total"] / 300) * 100

    df["Grade"] = df["Percentage"].apply(calculate_grade)

    print("\n========== STUDENT RESULTS ==========")

    for _, row in df.iterrows():

        print(
            f"{row['Name']} | "
            f"{row['Total']:g} | "
            f"{row['Percentage']:.2f}% | "
            f"{row['Grade']}"
        )

    return df


# Calculate class statistics
def calculate_statistics(df):

    if df.empty:
        print("No valid student records available.")
        return

    percentages = (df[subjects].sum(axis=1) / 300) * 100

    mean = np.mean(percentages)
    median = np.median(percentages)
    std_dev = np.std(percentages)

    print("\n========== CLASS STATISTICS ==========")

    print(f"Mean %: {mean:.2f}")
    print(f"Median %: {median:.2f}")
    print(f"Std Dev: {std_dev:.2f}")

    # Find topper
    topper_index = percentages.idxmax()

    topper_name = df.loc[topper_index, "Name"]
    topper_percentage = percentages.loc[topper_index]

    print(
        f"Topper: {topper_name} "
        f"({topper_percentage:.2f}%)"
    )

    # Subject-wise averages
    subject_averages = df[subjects].mean()

    print("\nSubject averages:")

    for subject, average in subject_averages.items():

        print(f"{subject} {average:.2f}")

    # Bar chart
    plt.figure(figsize=(8, 5))

    plt.bar(
        subject_averages.index,
        subject_averages.values,
        edgecolor="black"
    )

    plt.title("Subject-wise Average Marks")
    plt.xlabel("Subjects")
    plt.ylabel("Average Marks")
    plt.ylim(0, 100)
    plt.grid(axis="y", linestyle="--")
    plt.tight_layout()

    # Histogram
    plt.figure(figsize=(8, 5))

    plt.hist(
        percentages,
        bins=10,
        edgecolor="black"
    )

    plt.title("Student Percentage Distribution")
    plt.xlabel("Percentage")
    plt.ylabel("Number of Students")
    plt.tight_layout()

    print("Bar chart displayed")
    print("Histogram displayed")

    plt.show()


# Main program
def main():

    print("====================================")
    print("     STUDENT PERFORMANCE ANALYZER")
    print("====================================")

    df = load_data()

    if df.empty:
        return

    # Calculate results once
    result_df = calculate_results(df)

    while True:

        print("\n1. Result")
        print("2. Statistics")
        print("3. Exit")

        choice = input("Enter your choice: ").strip()

        if choice == "1":

            print("\nDisplaying student results:")

            for _, row in result_df.iterrows():

                print(
                    f"{row['Name']} | "
                    f"{row['Total']:g} | "
                    f"{row['Percentage']:.2f}% | "
                    f"{row['Grade']}"
                )

        elif choice == "2":

            calculate_statistics(df)

        elif choice == "3":

            print("Thank you for using Student Performance Analyzer!")
            break

        else:

            print("Invalid choice. Please enter 1, 2 or 3.")


# Run program
if __name__ == "__main__":
    main()