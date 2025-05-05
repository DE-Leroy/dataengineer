## Reading from PDF files

```python
import camelot

def extract_tables_from_pdf_camelot(pdf_path):
    """Extract tabular from PDF-files, using camelot.

    Args:
        pdf_path (str): Path to PDF-file.

    Returns:
        list: List of objects Table in camelot.
    """
    try:
        tables = camelot.read_pdf(pdf_path, pages='all')
        return tables
    except FileNotFoundError:
        print(f"Error: There is not file on path '{pdf_path}'.")
        return []
    except Exception as e:
        print(f"The error occurred: {e}")
        return []

if __name__ == "__main__":
    pdf_file_path = '2016.pdf'  # Change to yout file path
    extracted_tables = extract_tables_from_pdf_camelot(pdf_file_path)
```

## Extract and cleaning data

```python
import pandas as pd

df = extracted_tables[0].df

columns = [
    'Model Number',
    'Item Description',
    'Detail Description',
    'Overall Width',
    'Overall Depth',
    'Overall Height',
    'Arm Height',
    'Seat Height',
    'Seat Depth',
    'Inside Width',
    'Weight (Wrap)',
    'Weight (Carton)',
    'Cubic Feet',
    'COM Price',
    'COM Yardage',
    'Year'
]

df_answer = pd.DataFrame([], columns=columns)

for page in extracted_tables:
    df = page.df
    for i in range(3, len(df.columns)):
        row_list = [
            df[i].iloc[0],
            ', '.join(df[i].iloc[1].split('\n')),
            ', '.join(df[i].iloc[2].split('\n')),
            df[i].iloc[3],
            df[i].iloc[4],
            df[i].iloc[5],
            df[i].iloc[6],
            df[i].iloc[7],
            df[i].iloc[8],
            df[i].iloc[9]
        ]
        wrap_carton_cube = df[i].iloc[11].split()
        try:
            wrap_carton_cube.remove('/')
        except:
            pass
        wrap_carton_cube = [i.strip('/') for i in wrap_carton_cube]
        if len(wrap_carton_cube) == 0:
            row_list.append('')
            row_list.append('')
            row_list.append('')
        elif len(wrap_carton_cube) != 3:
            row_list.append('')
            row_list.append('')
            row_list.append(wrap_carton_cube[0])
        else:
            row_list.append(wrap_carton_cube[0])
            row_list.append(wrap_carton_cube[1])
            row_list.append(wrap_carton_cube[2])
        row_list.append(df[i].iloc[16])
        row_list.append(df[i].iloc[31])
        row_list.append(pdf_file_path.split('.')[0])

        df_answer = pd.concat([df_answer, pd.DataFrame([row_list], columns=columns)])
```

## Save data into CSV report

```python
df_answer.reset_index(drop=True, inplace=True)
df_answer[df_answer['COM Price'] != ''].to_csv('part_job.csv', index=False)
```