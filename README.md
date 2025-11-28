using System;
using System.Collections.Generic;
using System.Linq;

// -----------------------------------------------------------------
// Class 1: Employee - Demonstrates Properties, Fields, and Statics
// -----------------------------------------------------------------
public class Employee
{
    // 4. Static Field - Used for generating unique, sequential IDs across all employees
    private static int _nextId = 1001;

    // 4. Static Property - Tracks total objects created
    // This is read-only (private set), ensuring only the class controls the count.
    public static int TotalEmployeesCreated { get; private set; } = 0;

    // 2. Private Field (backing field)
    private double _salary;
    private string _employeeId;

    // 3. Auto-Implemented Properties (simplest form)
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public string Role { get; set; }

    // 3. Property Accessor: Read-Only Property (set is private)
    public string EmployeeId
    {
        get { return _employeeId; }
        private set { _employeeId = value; }
    }

    // 3. Property Accessors: Full Property with Validation Logic in the 'set'
    public double Salary
    {
        get { return _salary; }
        set
        {
            // Validation: Salary must be non-negative
            if (value < 0)
            {
                Console.WriteLine($"[ERROR] Invalid salary attempt for {FirstName} {LastName}. Salary must be non-negative. Value remains {_salary:C0}.");
            }
            else
            {
                _salary = value;
            }
        }
    }

    // 3. Constructor Overloading & Chaining (Parameterless constructor)
    // Calls the two-parameter constructor with default values
    public Employee() : this("Unknown", "Employee", 0.0)
    {
        // Object Initializers allow us to set properties without needing multiple constructors.
    }

    // 3. Constructor Overloading & Chaining (Two-parameter constructor)
    // Calls the main three-parameter constructor
    public Employee(string firstName, string lastName) : this(firstName, lastName, 0.0)
    {
    }

    // 3. Constructor (Main) - Initializes all essential values
    public Employee(string firstName, string lastName, double salary)
    {
        // 6. Class Design Principle: Encapsulation applied through Properties
        this.FirstName = firstName;
        this.LastName = lastName;
        this.Salary = salary; // Uses the 'set' accessor for initial validation

        // Set the unique ID using the static generator
        this.EmployeeId = GenerateId();

        // Update static counter
        TotalEmployeesCreated++;
    }

    // 4. Static Method - Accessible via the class name (Employee.GenerateId())
    private static string GenerateId()
    {
        string id = $"EMP{_nextId:D4}";
        _nextId++;
        return id;
    }

    public void DisplayDetails()
    {
        Console.WriteLine($"\n--- {EmployeeId} ({Role}) ---");
        Console.WriteLine($"Name: {FirstName} {LastName}");
        Console.WriteLine($"Salary: {Salary:C0}");
    }
}

// -----------------------------------------------------------------
// Class 2: Department - Demonstrates Collection Management
// -----------------------------------------------------------------
public class Department
{
    // Auto-implemented properties
    public string Name { get; set; }
    public int MaxEmployees { get; set; }

    // Collection property (5. Collection Initializer uses this List)
    public List<Employee> Employees { get; set; } = new List<Employee>();

    // Constructor
    public Department(string name, int maxEmployees)
    {
        this.Name = name;
        this.MaxEmployees = maxEmployees;
    }

    public void AddEmployee(Employee emp)
    {
        if (Employees.Count < MaxEmployees)
        {
            Employees.Add(emp);
            Console.WriteLine($"[SUCCESS] {emp.FirstName} added to {Name}.");
        }
        else
        {
            Console.WriteLine($"[FAIL] {Name} is full ({MaxEmployees} max). Cannot add {emp.FirstName}.");
        }
    }

    // Innovative Feature: Calculates average salary using LINQ
    public double CalculateAverageSalary()
    {
        if (Employees.Count == 0) return 0;
        return Employees.Average(e => e.Salary);
    }
}

// -----------------------------------------------------------------
// Class 3: Program - Main execution
// -----------------------------------------------------------------
public class Program
{
    public static void Main(string[] args)
    {
        Console.WriteLine("==================================================");
        Console.WriteLine(" C# EMPLOYEE MANAGEMENT SYSTEM DEMO");
        Console.WriteLine("==================================================");

        // --- 1. Object Initializers Demonstration ---
        Console.WriteLine("\n--- 1. Creating Employee Objects using Initializers ---");

        // Employee 1: Full constructor, then setting Role with Object Initializer
        Employee manager = new Employee("Alice", "Johnson", 95000.00)
        {
            Role = "Manager"
        };
        manager.DisplayDetails();

        // Employee 2: Overloaded constructor, then setting properties
        Employee developer = new Employee("Bob", "Smith")
        {
            Role = "Senior Developer",
            Salary = 75000.00 // Setting salary via property 'set' accessor
        };
        developer.DisplayDetails();

        // Employee 3: Parameterless constructor and Object Initializer for everything
        Employee intern = new Employee
        {
            FirstName = "Charlie",
            LastName = "Brown",
            Role = "Intern",
            Salary = 30000.00
        };
        intern.DisplayDetails();
        
        // Demonstrating Property 'set' Accessor Validation
        Console.WriteLine("\n--- 2. Property Validation Demo ---");
        developer.Salary = -500.00; // This will trigger the validation message and deny the change
        developer.DisplayDetails(); // Salary remains unchanged at 75000.00


        // --- 3. Collection Initializers & Department Setup ---
        Console.WriteLine("\n--- 3. Setting Up Department with Collection Initializers ---");
        
        Department engineering = new Department("Engineering", 3)
        {
            // Collection Initializer for the Employees List
            Employees = new List<Employee>
            {
                manager,
                developer,
                intern
            }
        };
        
        Console.WriteLine($"Department '{engineering.Name}' initialized with {engineering.Employees.Count} employees.");

        // --- 4. Interactive User Input (Simulation of adding a new employee) ---
        
        Console.WriteLine("\n--- 4. Interactive Employee Creation (Input Required) ---");
        
        Console.Write("Enter New Employee First Name: ");
        string newFName = Console.ReadLine() ?? "David";

        Console.Write("Enter New Employee Last Name: ");
        string newLName = Console.ReadLine() ?? "Lee";

        Console.Write("Enter New Employee Role: ");
        string newRole = Console.ReadLine() ?? "Junior Analyst";

        Console.Write("Enter New Employee Salary: ");
        double newSalary;
        // Simple TryParse for robust input handling
        if (!double.TryParse(Console.ReadLine(), out newSalary))
        {
            newSalary = 40000.00; // Default if invalid
            Console.WriteLine("[INFO] Invalid input or empty. Defaulting salary to 40000.");
        }

        // Employee 4: Created interactively
        Employee newEmployee = new Employee(newFName, newLName, newSalary)
        {
            Role = newRole // Uses Object Initializer syntax
        };
        newEmployee.DisplayDetails();

        // Try to add the new employee to the department (which is currently full: MaxEmployees=3, Count=3)
        Console.WriteLine($"\nAttempting to add {newEmployee.FirstName} to the department (Max: {engineering.MaxEmployees})...");
        engineering.AddEmployee(newEmployee); // This should fail

        // Increase the limit and try again
        Console.WriteLine("\nIncreasing department capacity (MaxEmployees = 5)...");
        engineering.MaxEmployees = 5;
        engineering.AddEmployee(newEmployee); // This should succeed now

        // --- 5. Final Summary & Static Member Check ---
        
        Console.WriteLine("\n--- 5. Final Department Summary ---");
        Console.WriteLine($"Department: {engineering.Name}");
        Console.WriteLine($"Current Employees: {engineering.Employees.Count}/{engineering.MaxEmployees}");
        Console.WriteLine($"Average Salary: {engineering.CalculateAverageSalary():C2}");

        Console.WriteLine("\n--- Static Member Check ---");
        // Accessing the static property
        Console.WriteLine($"Total Employee objects ever created: {Employee.TotalEmployeesCreated}");
        
        Console.WriteLine("\nPress any key to exit...");
        Console.ReadKey();
    }
}
