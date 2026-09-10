#include <stdio.h>
#include <string.h>

#define MAX 6

struct Medicine
{
    char code[10];
    char name[30];

    int stock;
    int dailyReq;
    int minStock;
    int expiryDays;
    int essentiality;

    float coverage;
    char condition[30];
    float priorityScore;
};

typedef struct Medicine Medicine;


/* Calculate coverage, condition and priority score */
void calculate_status(Medicine *m)
{
    m->coverage = (float)m->stock / m->dailyReq;

    if (m->stock < m->minStock && m->expiryDays <= 25)
    {
        strcpy(m->condition, "Critical Condition");
    }
    else if (m->stock < m->minStock && m->essentiality == 3)
    {
        strcpy(m->condition, "Urgent Reorder");
    }
    else if ((m->stock < m->minStock && m->essentiality < 3) ||
             m->coverage < 5.0)
    {
        strcpy(m->condition, "Reorder Required");
    }
    else if (m->stock >= m->minStock && m->expiryDays <= 25)
    {
        strcpy(m->condition, "Expiry Attention");
    }
    else
    {
        strcpy(m->condition, "Sufficient Stock");
    }

    m->priorityScore =
        (m->essentiality * 30.0) +
        ((m->minStock - m->stock) * 0.5) -
        (m->expiryDays * 0.8);
}


/* Analyze all medicines */
void analyze_all(Medicine arr[], int n)
{
    int i;

    for (i = 0; i < n; i++)
    {
        calculate_status(&arr[i]);
    }
}


/* Display medicine analysis */
void display_analysis(Medicine arr[], int n)
{
    int i;

    printf("\n========== MEDICINE ANALYSIS ==========\n");

    printf("%-8s %-15s %-10s %-20s %-10s\n",
           "Code", "Name", "Coverage", "Condition", "Priority");

    printf("---------------------------------------------------------------\n");

    for (i = 0; i < n; i++)
    {
        printf("%-8s %-15s %-10.1f %-20s %-10.1f\n",
               arr[i].code,
               arr[i].name,
               arr[i].coverage,
               arr[i].condition,
               arr[i].priorityScore);
    }
}


/* Search medicine */
int search_medicine(Medicine arr[], int n, char code[])
{
    int i;

    for (i = 0; i < n; i++)
    {
        if (strcmp(arr[i].code, code) == 0)
        {
            return i;
        }
    }

    return -1;
}


/* Update stock */
void update_stock(Medicine arr[], int n)
{
    char code[10];
    int index;
    int choice;
    int amount;

    printf("\nEnter Medicine Code: ");
    scanf("%s", code);

    index = search_medicine(arr, n, code);

    if (index == -1)
    {
        printf("Medicine Not Found!\n");
        return;
    }

    printf("\n1. Add Stock\n");
    printf("2. Issue Stock\n");

    printf("Enter Choice: ");
    scanf("%d", &choice);

    printf("Enter Amount: ");
    scanf("%d", &amount);

    if (choice == 1)
    {
        arr[index].stock =
            arr[index].stock + amount;

        calculate_status(&arr[index]);

        printf("Stock Added Successfully!\n");
    }

    else if (choice == 2)
    {
        if (amount > arr[index].stock)
        {
            printf("Error: Insufficient Stock!\n");
        }

        else
        {
            arr[index].stock =
                arr[index].stock - amount;

            calculate_status(&arr[index]);

            printf("Stock Issued Successfully!\n");
        }
    }

    else
    {
        printf("Invalid Choice!\n");
    }
}


/* Sort by priority score */
void sort_by_priority(Medicine arr[], int n)
{
    int i, j, maxIndex;
    Medicine temp;

    for (i = 0; i < n - 1; i++)
    {
        maxIndex = i;

        for (j = i + 1; j < n; j++)
        {
            if (arr[j].priorityScore >
                arr[maxIndex].priorityScore)
            {
                maxIndex = j;
            }

            else if (arr[j].priorityScore ==
                     arr[maxIndex].priorityScore)
            {
                if (arr[j].essentiality >
                    arr[maxIndex].essentiality)
                {
                    maxIndex = j;
                }

                else if (arr[j].essentiality ==
                         arr[maxIndex].essentiality)
                {
                    if (arr[j].expiryDays <
                        arr[maxIndex].expiryDays)
                    {
                        maxIndex = j;
                    }

                    else if (arr[j].expiryDays ==
                             arr[maxIndex].expiryDays)
                    {
                        if (strcmp(arr[j].code,
                                   arr[maxIndex].code) < 0)
                        {
                            maxIndex = j;
                        }
                    }
                }
            }
        }

        temp = arr[i];
        arr[i] = arr[maxIndex];
        arr[maxIndex] = temp;
    }
}


/* Generate report file */
void generate_report_file(Medicine arr[], int n)
{
    FILE *fp;
    int i;

    fp = fopen("stock_report.txt", "w");

    if (fp == NULL)
    {
        printf("File Cannot Be Created!\n");
        return;
    }

    fprintf(fp,
            "MEDICINE STOCK AND EXPIRY MANAGEMENT REPORT\n\n");

    fprintf(fp,
            "%-8s %-15s %-10s %-20s %-10s\n",
            "Code", "Name", "Coverage",
            "Condition", "Priority");

    fprintf(fp,
            "---------------------------------------------------------------\n");

    for (i = 0; i < n; i++)
    {
        fprintf(fp,
                "%-8s %-15s %-10.1f %-20s %-10.1f\n",
                arr[i].code,
                arr[i].name,
                arr[i].coverage,
                arr[i].condition,
                arr[i].priorityScore);
    }

    fclose(fp);

    printf("Report Saved Successfully!\n");
}


/* Main Function */
int main()
{
    Medicine arr[MAX] =
    {
        {"MED01", "Oral Saline", 120, 35, 80, 45, 3},
        {"MED02", "Paracetamol", 300, 40, 100, 120, 2},
        {"MED03", "Insulin", 60, 12, 50, 30, 3},
        {"MED04", "Amoxicillin", 200, 25, 80, 20, 3},
        {"MED05", "Antacid", 250, 18, 60, 15, 1},
        {"MED06", "Cetirizine", 180, 15, 50, 90, 1}
    };

    int n = MAX;
    int choice;

    char code[10];
    int index;

    analyze_all(arr, n);

    do
    {
        printf("\n\n===== MEDICINE STOCK MANAGEMENT SYSTEM =====\n");

        printf("1. View Analysis\n");
        printf("2. Search Record\n");
        printf("3. Update Stock\n");
        printf("4. Order by Priority\n");
        printf("5. Export File Report\n");
        printf("6. Exit\n");

        printf("\nEnter Your Choice: ");
        scanf("%d", &choice);

        switch (choice)
        {
            case 1:
                display_analysis(arr, n);
                break;


            case 2:

                printf("Enter Medicine Code: ");
                scanf("%s", code);

                index = search_medicine(arr, n, code);

                if (index == -1)
                {
                    printf("Medicine Not Found!\n");
                }

                else
                {
                    printf("\nCode: %s\n", arr[index].code);
                    printf("Name: %s\n", arr[index].name);
                    printf("Stock: %d\n", arr[index].stock);
                    printf("Daily Requirement: %d\n",
                           arr[index].dailyReq);
                    printf("Minimum Stock: %d\n",
                           arr[index].minStock);
                    printf("Expiry Days: %d\n",
                           arr[index].expiryDays);
                    printf("Essentiality: %d\n",
                           arr[index].essentiality);
                    printf("Coverage: %.1f\n",
                           arr[index].coverage);
                    printf("Condition: %s\n",
                           arr[index].condition);
                    printf("Priority Score: %.1f\n",
                           arr[index].priorityScore);
                }

                break;


            case 3:
                update_stock(arr, n);
                break;


            case 4:
                sort_by_priority(arr, n);
                display_analysis(arr, n);
                break;


            case 5:
                generate_report_file(arr, n);
                break;


            case 6:
                printf("Program Exited Successfully!\n");
                break;


            default:
                printf("Invalid Choice!\n");
        }

    }
    while (choice != 6);

    return 0;
}
