using System;
using System.Collections.Generic;
using System.IO;

namespace DebtTracker
{
    class Debt
    {
        public string DebtorName { get; set; }
        public decimal Amount { get; set; }
        public DateTime DateOfDebt { get; set; }
        public string Description { get; set; }

        public Debt(string debtorName, decimal amount, DateTime dateOfDebt, string description)
        {
            DebtorName = debtorName;
            Amount = amount;
            DateOfDebt = dateOfDebt;
            Description = description;
        }
    }

    class Program
    {
        static List<Debt> debts = new List<Debt>();
        static string dataFile = "debts.txt";

        static void Main(string[] args)
        {
            LoadDebts();

            Console.WriteLine("Добро пожаловать в систему учета долгов!");

            while (true)
            {
                Console.WriteLine("\nВыберите действие:");
                Console.WriteLine("1. Добавить долг");
                Console.WriteLine("2. Отобразить долги");
                Console.WriteLine("3. Оплатить долг");
                Console.WriteLine("4. Выход");

                int choice = GetChoice();

                switch (choice)
                {
                    case 1:
                        AddDebt();
                        break;
                    case 2:
                        DisplayDebts();
                        break;
                    case 3:
                        PayDebt();
                        break;
                    case 4:
                        SaveDebts();
                        Console.WriteLine("До свидания!");
                        return;
                    default:
                        Console.WriteLine("Неверный выбор.");
                        break;
                }
            }
        }

        static int GetChoice()
        {
            while (true)
            {
                if (int.TryParse(Console.ReadLine(), out int choice))
                {
                    return choice;
                }
                else
                {
                    Console.WriteLine("Неверный ввод. Пожалуйста, введите число.");
                }
            }
        }

        static void AddDebt()
        {
            Console.WriteLine("Введите имя должника:");
            string debtorName = Console.ReadLine();

            Console.WriteLine("Введите сумму долга:");
            decimal amount = GetDecimalInput();

            Console.WriteLine("Введите дату возникновения долга (дд.мм.гггг):");
            DateTime dateOfDebt = GetDateInput();

            Console.WriteLine("Введите описание долга (необязательно):");
            string description = Console.ReadLine();

            debts.Add(new Debt(debtorName, amount, dateOfDebt, description));
            Console.WriteLine("Долг добавлен!");
        }

        static decimal GetDecimalInput()
        {
            while (true)
            {
                if (decimal.TryParse(Console.ReadLine(), out decimal value))
                {
                    return value;
                }
                else
                {
                    Console.WriteLine("Неверный ввод. Пожалуйста, введите число.");
                }
            }
        }

        static DateTime GetDateInput()
        {
            while (true)
            {
                if (DateTime.TryParseExact(Console.ReadLine(), "dd.MM.yyyy", null, System.Globalization.DateTimeStyles.None, out DateTime date))
                {
                    return date;
                }
                else
                {
                    Console.WriteLine("Неверный формат даты. Пожалуйста, введите дату в формате дд.мм.гггг.");
                }
            }
        }

        static void DisplayDebts()
        {
            if (debts.Count == 0)
            {
                Console.WriteLine("Нет долгов.");
            }
            else
            {
                Console.WriteLine("\nСписок долгов:");
                foreach (Debt debt in debts)
                {
                    Console.WriteLine($"Должник: {debt.DebtorName}, Сумма: {debt.Amount:C}, Дата: {debt.DateOfDebt:dd.MM.yyyy}, Описание: {debt.Description}");
                }
            }
        }

        static void PayDebt()
        {
            if (debts.Count == 0)
            {
                Console.WriteLine("Нет долгов.");
                return;
            }

            DisplayDebts();

            Console.WriteLine("Введите имя должника, чей долг вы хотите оплатить:");
            string debtorName = Console.ReadLine();

            Debt debtToPay = debts.Find(debt => debt.DebtorName == debtorName);

            if (debtToPay == null)
            {
                Console.WriteLine("Долг с таким именем не найден.");
            }
            else
            {
                Console.WriteLine($"Введите сумму оплаты для {debtToPay.DebtorName}:");
                decimal paymentAmount = GetDecimalInput();

                if (paymentAmount > debtToPay.Amount)
                {
                    Console.WriteLine("Сумма оплаты больше, чем сумма долга.");
                }
                else
                {
                    debtToPay.Amount -= paymentAmount;
                    Console.WriteLine($"Долг {debtToPay.DebtorName} частично оплачен.");

                    if (debtToPay.Amount == 0)
                    {
                        debts.Remove(debtToPay);
                        Console.WriteLine($"Долг {debtToPay.DebtorName} полностью оплачен.");
                    }
                }
            }
        }

        static void SaveDebts()
        {
            using (StreamWriter writer = new StreamWriter(dataFile))
            {
                foreach (Debt debt in debts)
                {
                    writer.WriteLine($"{debt.DebtorName},{debt.Amount},{debt.DateOfDebt:yyyy-MM-dd},{debt.Description}");
                }
            }
        }

        static void LoadDebts()
        {
            if (File.Exists(dataFile))
            {
                using (StreamReader reader = new StreamReader(dataFile))
                {
                    string line;
                    while ((line = reader.ReadLine()) != null)
                    {
                        string[] parts = line.Split(',');
                        debts.Add(new Debt(parts[0], decimal.Parse(parts[1]), DateTime.Parse(parts[2]), parts[3]));
                    }
                }
            }
        }
    }
}
