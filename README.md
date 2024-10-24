-- Create database
CREATE DATABASE HospitalManagement;

-- Use database
USE HospitalManagement;

-- Create tables

CREATE TABLE Patients (
  PatientID INT PRIMARY KEY,
  Name VARCHAR(255) NOT NULL,
  DateOfBirth DATE NOT NULL,
  ContactNumber VARCHAR(20) NOT NULL,
  Address VARCHAR(255) NOT NULL
);

CREATE TABLE Doctors (
  DoctorID INT PRIMARY KEY,
  Name VARCHAR(255) NOT NULL,
  Specialty VARCHAR(100) NOT NULL,
  ContactNumber VARCHAR(20) NOT NULL
);

CREATE TABLE Appointments (
  AppointmentID INT PRIMARY KEY,
  PatientID INT NOT NULL,
  DoctorID INT NOT NULL,
  AppointmentDate DATE NOT NULL,
  AppointmentTime TIME NOT NULL,
  FOREIGN KEY (PatientID) REFERENCES Patients(PatientID),
  FOREIGN KEY (DoctorID) REFERENCES Doctors(DoctorID)
);

CREATE TABLE Medications (
  MedicationID INT PRIMARY KEY,
  Name VARCHAR(100) NOT NULL,
  Description VARCHAR(255) NOT NULL
);

CREATE TABLE Prescriptions (
  PrescriptionID INT PRIMARY KEY,
  PatientID INT NOT NULL,
  DoctorID INT NOT NULL,
  MedicationID INT NOT NULL,
  Dosage VARCHAR(100) NOT NULL,
  Duration VARCHAR(100) NOT NULL,
  FOREIGN KEY (PatientID) REFERENCES Patients(PatientID),
  FOREIGN KEY (DoctorID) REFERENCES Doctors(DoctorID),
  FOREIGN KEY (MedicationID) REFERENCES Medications(MedicationID)
);

CREATE TABLE Rooms (
  RoomID INT PRIMARY KEY,
  RoomType VARCHAR(100) NOT NULL,
  Availability VARCHAR(10) NOT NULL
);

CREATE TABLE Admissions (
  AdmissionID INT PRIMARY KEY,
  PatientID INT NOT NULL,
  RoomID INT NOT NULL,
  AdmissionDate DATE NOT NULL,
  DischargeDate DATE,
  FOREIGN KEY (PatientID) REFERENCES Patients(PatientID),
  FOREIGN KEY (RoomID) REFERENCES Rooms(RoomID)
);

CREATE TABLE Payments (
  PaymentID INT PRIMARY KEY,
  PatientID INT NOT NULL,
  PaymentDate DATE NOT NULL,
  Amount DECIMAL(10, 2) NOT NULL,
  FOREIGN KEY (PatientID) REFERENCES Patients(PatientID)
);

-- Insert sample data

INSERT INTO Patients (PatientID, Name, DateOfBirth, ContactNumber, Address)
VALUES
(1, 'John Doe', '1990-01-01', '1234567890', '123 Main St'),
(2, 'Jane Doe', '1995-06-01', '9876543210', '456 Elm St');

INSERT INTO Doctors (DoctorID, Name, Specialty, ContactNumber)
VALUES
(1, 'Dr. Smith', 'Cardiology', '5551234567'),
(2, 'Dr. Johnson', 'Oncology', '5559876543');

INSERT INTO Appointments (AppointmentID, PatientID, DoctorID, AppointmentDate, AppointmentTime)
VALUES
(1, 1, 1, '2024-10-13', '10:00:00'),
(2, 2, 2, '2024-10-14', '11:00:00');

INSERT INTO Medications (MedicationID, Name, Description)
VALUES
(1, 'Aspirin', 'Pain reliever'),
(2, 'Ibuprofen', 'Anti-inflammatory');

INSERT INTO Prescriptions (PrescriptionID, PatientID, DoctorID, MedicationID, Dosage, Duration)
VALUES
(1, 1, 1, 1, 'Take 2 tablets daily', '5 days'),
(2, 2, 2, 2, 'Take 1 tablet daily', '7 days');

INSERT INTO Rooms (RoomID, RoomType, Availability)
VALUES
(1, 'Private', 'Yes'),
(2, 'Shared', 'Yes');

INSERT INTO Admissions (AdmissionID, PatientID, RoomID, AdmissionDate, DischargeDate)
VALUES
(1, 1, 1, '2024-10-13', NULL),
(2, 2, 2, '2024-10-14', '2024-10-21');

INSERT INTO Payments (PaymentID, PatientID, PaymentDate, Amount)
VALUES
(1, 1, '2024-10-13', 100.00),
(2, 2, '2024-10-14', 200.00);

-- Retrieve all patients
SELECT * FROM Patients;

-- Retrieve all doctors
SELECT * FROM Doctors;

-- Retrieve all appointments
SELECT * FROM Appointments;

-- Retrieve all medications
SELECT * FROM Medications;

-- Retrieve all prescriptions
SELECT * FROM Prescriptions;

-- Retrieve all rooms
SELECT * FROM Rooms;

-- Retrieve all admissions
SELECT * FROM Admissions;

-- Retrieve all payments
SELECT * FROM Payments;

-- Retrieve patients by doctor
SELECT p.Name, d.Name AS DoctorName 
FROM Patients p
JOIN Appointments a ON p.PatientID = a.PatientID
JOIN Doctors d ON a.DoctorID = d.DoctorID;

-- Retrieve prescriptions for a specific patient
SELECT pr.*, m.Name AS MedicationName 
FROM Prescriptions pr
JOIN Medications m ON pr.MedicationID = m.MedicationID
WHERE pr.PatientID = 1;

-- Retrieve admissions by room type
SELECT a.*, r.RoomType 
FROM Admissions a
JOIN Rooms r ON a.RoomID = r.RoomID;

-- Retrieve payments by patient
SELECT p.Name, py.* 
FROM Patients p
JOIN Payments py ON p.PatientID = py.PatientID;

-- Count the number of patients
SELECT COUNT(*) AS TotalPatients FROM Patients;

-- List all doctors and the number of patients they've seen
SELECT d.Name AS DoctorName, COUNT(a.PatientID) AS PatientCount
FROM Doctors d
LEFT JOIN Appointments a ON d.DoctorID = a.DoctorID
GROUP BY d.Name;

-- Find the most common specialty among doctors
SELECT Specialty, COUNT(*) AS DoctorCount
FROM Doctors
GROUP BY Specialty
ORDER BY DoctorCount DESC;

-- Get the details of appointments in the upcoming week
SELECT a.*, p.Name AS PatientName, d.Name AS DoctorName 
FROM Appointments a
JOIN Patients p ON a.PatientID = p.PatientID
JOIN Doctors d ON a.DoctorID = d.DoctorID
WHERE a.AppointmentDate BETWEEN CURDATE() AND DATE_ADD(CURDATE(), INTERVAL 7 DAY);

-- Find the number of appointments each doctor has
SELECT d.Name AS DoctorName, COUNT(a.AppointmentID) AS AppointmentCount
FROM Doctors d
LEFT JOIN Appointments a ON d.DoctorID = a.DoctorID
GROUP BY d.Name;

-- Retrieve patients who have been prescribed a specific medication
SELECT p.Name, m.Name AS MedicationName
FROM Patients p
JOIN Prescriptions pr ON p.PatientID = pr.PatientID
JOIN Medications m ON pr.MedicationID = m.MedicationID
WHERE m.Name = 'Aspirin';

-- Get the total amount of payments made by each patient
SELECT p.Name, SUM(py.Amount) AS TotalPaid
FROM Patients p
JOIN Payments py ON p.PatientID = py.PatientID
GROUP BY p.Name;

-- List all rooms and their current availability status
SELECT RoomType, COUNT(*) AS AvailableRooms
FROM Rooms
WHERE Availability = 'Yes'
GROUP BY RoomType;

-- Find the average length of hospital stay
SELECT AVG(DATEDIFF(DischargeDate, AdmissionDate)) AS AvgStayLength
FROM Admissions
WHERE DischargeDate IS NOT NULL;

-- Get patients who haven't had an appointment in the last year
SELECT p.Name, MAX(a.AppointmentDate) AS LastAppointment
FROM Patients p
LEFT JOIN Appointments a ON p.PatientID = a.PatientID
GROUP BY p.Name
HAVING LastAppointment < DATE_SUB(CURDATE(), INTERVAL 1 YEAR) OR LastAppointment IS NULL;

-- Retrieve the contact information for patients in a specific room type
SELECT p.Name, p.ContactNumber, r.RoomType
FROM Patients p
JOIN Admissions a ON p.PatientID = a.PatientID
JOIN Rooms r ON a.RoomID = r.RoomID
WHERE r.RoomType = 'Private';

-- List the number of admissions per room type
SELECT r.RoomType, COUNT(a.AdmissionID) AS AdmissionCount
FROM Rooms r
LEFT JOIN Admissions a ON r.RoomID = a.RoomID
GROUP BY r.RoomType;

-- Get all patients who have been admitted more than once
SELECT p.Name, COUNT(a.AdmissionID) AS AdmissionCount
FROM Patients p
JOIN Admissions a ON p.PatientID = a.PatientID
GROUP BY p.Name
HAVING AdmissionCount > 1;

-- Retrieve the latest prescription for each patient
SELECT p.Name, pr.*
FROM Patients p
JOIN Prescriptions pr ON p.PatientID = pr.PatientID
WHERE (pr.PatientID, pr.PrescriptionID) IN (
  SELECT PatientID, MAX(PrescriptionID)
  FROM Prescriptions
  GROUP BY PatientID
);

-- Find doctors who have the most active appointments
SELECT d.Name, COUNT(a.AppointmentID) AS ActiveAppointments
FROM Doctors d
JOIN Appointments a ON d.DoctorID = a.DoctorID
WHERE a.AppointmentDate > CURDATE()
GROUP BY d.Name
ORDER BY ActiveAppointments DESC;

-- Retrieve details of patients with pending payments
SELECT p.Name, py.PaymentDate, py.Amount
FROM Patients p
JOIN Payments py ON p.PatientID = py.PatientID
WHERE py.Amount > 0
ORDER BY py.PaymentDate DESC;

-- Get the sum of payments received per day
SELECT PaymentDate, SUM(Amount) AS TotalAmount
FROM Payments
GROUP BY PaymentDate
ORDER BY PaymentDate DESC;

-- List all patients who have the same doctor for different specialties
SELECT p.Name, d.Name AS DoctorName, d.Specialty
FROM Patients p
JOIN Appointments a ON p.PatientID = a.PatientID
JOIN Doctors d ON a.DoctorID = d.DoctorID
GROUP BY p.Name, d.Name, d.Specialty
HAVING COUNT(DISTINCT d.Specialty) > 1;
# Hospital-Management-SQl-project
