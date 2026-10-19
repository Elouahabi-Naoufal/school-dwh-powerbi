<a id="readme-top"></a>

[![Contributors][contributors-shield]][contributors-url]
[![Forks][forks-shield]][forks-url]
[![Stargazers][stars-shield]][stars-url]
[![Issues][issues-shield]][issues-url]
[![LinkedIn][linkedin-shield]][linkedin-url]

<br />
<div align="center">
  <a href="https://github.com/Elouahabi-Naoufal/school-data-warehouse-powerbi">
    <h3 align="center">School Data Warehouse and Power BI</h3>
  </a>

  <p align="center">
    A decision-support warehouse for a school: 27 source tables, a dbt project on Snowflake, and a Power BI reporting layer.
    <br />
    <a href="https://github.com/Elouahabi-Naoufal/school-data-warehouse-powerbi"><strong>Explore the repository »</strong></a>
    <br />
    <br />
    <a href="https://github.com/Elouahabi-Naoufal/school-data-warehouse-powerbi/issues/new?labels=bug">Report Bug</a>
    &middot;
    <a href="https://github.com/Elouahabi-Naoufal/school-data-warehouse-powerbi/issues/new?labels=enhancement">Request Feature</a>
  </p>
</div>

<details>
  <summary>Table of Contents</summary>
  <ol>
    <li><a href="#about-the-project">About The Project</a>
      <ul>
        <li><a href="#data-flow">Data flow</a></li>
        <li><a href="#repository-layout">Repository layout</a></li>
        <li><a href="#built-with">Built With</a></li>
      </ul>
    </li>
    <li><a href="#getting-started">Getting Started</a>
      <ul>
        <li><a href="#prerequisites">Prerequisites</a></li>
        <li><a href="#installation">Installation</a></li>
      </ul>
    </li>
    <li><a href="#usage">Usage</a></li>
    <li><a href="#roadmap">Roadmap</a></li>
    <li><a href="#contributing">Contributing</a></li>
    <li><a href="#license">License</a></li>
    <li><a href="#contact">Contact</a></li>
    <li><a href="#acknowledgments">Acknowledgments</a></li>
  </ol>
</details>

## About The Project

This project builds a decision-support system for a school, from raw operational extracts to
analytical marts. The source model covers students, enrolments, modules, grades, attendance,
teaching staff, finance, student life, satisfaction surveys, professional integration and
administrative activity — 27 tables in total.

The repository has three working parts:

* a Colab notebook that profiles and cleans the source workbook;
* a Python loader that writes the CSV extracts into `DW_ECOLE.RAW` on Snowflake;
* a dbt project (`dw_ecole`) with 26 staging views and ten analytical marts built on top of that
  raw schema.

The Power BI report consumes the marts. The `.pbix` file itself is not committed in this
repository; what is committed is the warehouse and the model documentation it is meant to feed.

### Data flow

```text
Excel workbook (27 sheets)
        |
        v
notebooks/01_EDA_and_Cleaning.ipynb     profiling, cleaning, feature checks
        |
        v
data/csv/*.csv (27 files)
        |
        v
scripts/upload_to_snowflake.py          -> Snowflake DW_ECOLE.RAW
        |
        v
dbt dw_ecole
  staging/      26 views                 -> STAGING schema
  mart_*/       fact + dimension tables  -> one schema per mart
        |
        v
Power BI report (not included in this repository)
```

The marts are: inscriptions, academic performance, graduation and professional integration,
teachers, administration, student workload, student life, finance, satisfaction, and an executive
KPI summary.

### Repository layout

```text
.
├── data/csv/                 # 27 CSV extracts of the source workbook
├── notebooks/
│   └── 01_EDA_and_Cleaning.ipynb
├── scripts/
│   └── upload_to_snowflake.py
├── dw_ecole/                 # dbt project
│   ├── dbt_project.yml
│   ├── packages.yml          # dbt-labs/dbt_utils
│   └── models/
│       ├── sources.yml       # DW_ECOLE.RAW source declarations
│       ├── staging/          # 26 staging views
│       └── mart_*/           # fact and dimension models
├── dbt_files/                # early single-mart draft (inscriptions only)
└── docs/
    ├── Schema_Source.md      # source tables, column by column
    └── Tables_Schema_of_Marts.md
```

`dbt_files/` is an earlier draft kept for reference; the maintained project is `dw_ecole/`.

<p align="right">(<a href="#readme-top">back to top</a>)</p>

### Built With

* [![Python][Python-badge]][Python-url]
* [![Snowflake][Snowflake-badge]][Snowflake-url]
* [![dbt][dbt-badge]][dbt-url]
* [![Power BI][PowerBI-badge]][PowerBI-url]
* [![pandas][pandas-badge]][pandas-url]
* [![Jupyter][Jupyter-badge]][Jupyter-url]
* [![SQL][SQL-badge]][SQL-url]

<p align="right">(<a href="#readme-top">back to top</a>)</p>

## Getting Started

### Prerequisites

* Python 3.9+ and `pip`
* A Snowflake account with a warehouse, database and schema you can write to
* dbt with the Snowflake adapter
  ```sh
  pip install dbt-snowflake
  ```
* Power BI Desktop if you want to rebuild the report

### Installation

1. Clone the repo
   ```sh
   git clone https://github.com/Elouahabi-Naoufal/school-data-warehouse-powerbi.git
   cd school-data-warehouse-powerbi
   ```

2. Install the Python dependencies used by the notebook and the loader
   ```sh
   python -m venv .venv
   source .venv/bin/activate
   pip install pandas numpy matplotlib seaborn scipy missingno openpyxl snowflake-connector-python
   ```

3. Create a dbt profile named `dw_ecole` (the project expects it in `dbt_project.yml`). The
   repository does not ship a `profiles.yml`; keep yours outside version control.
   ```yaml
   dw_ecole:
     target: dev
     outputs:
       dev:
         type: snowflake
         account: <your-account>
         user: <your-user>
         password: <your-password>
         role: <your-role>
         database: DW_ECOLE
         warehouse: <your-warehouse>
         schema: RAW
   ```

<p align="right">(<a href="#readme-top">back to top</a>)</p>

## Usage

### 1. Profile and clean the source data

Open `notebooks/01_EDA_and_Cleaning.ipynb` in Jupyter or Google Colab. The first cells upload the
source workbook, then the notebook inspects every sheet, checks missing values and duplicates,
and prepares the extracts used downstream. The notebook was written for Colab; when running it
locally, replace the `google.colab` upload cell with a local path.

### 2. Load the CSV extracts into Snowflake

The loader creates `DW_ECOLE.RAW` if needed and writes every CSV with `write_pandas`, replacing
existing tables.

Note that `scripts/upload_to_snowflake.py` currently resolves its input as a `csv/` directory next
to the script, while the CSVs in this repository live in `data/csv/`. Either copy the files or
adjust `CSV_DIR` before running:

```sh
mkdir -p scripts/csv
cp data/csv/*.csv scripts/csv/
python scripts/upload_to_snowflake.py
```

The connection settings are currently hardcoded in the script. Move them to environment variables
or a local, untracked configuration file before using this against a real account.

### 3. Build the warehouse with dbt

```sh
cd dw_ecole
dbt deps
dbt run
dbt test      # when tests are added
```

`dbt deps` installs `dbt-labs/dbt_utils`. `dbt run` builds the 26 staging views first, then the
fact and dimension tables in the mart schemas.

### 4. Connect Power BI

Point Power BI Desktop at the `DW_ECOLE` database and the mart schemas. `docs/Schema_Source.md`
and `docs/Tables_Schema_of_Marts.md` describe the columns available on each side.

<p align="right">(<a href="#readme-top">back to top</a>)</p>

## Roadmap

- [x] Document the 27 source tables and the mart schemas
- [x] Clean and profile the source workbook
- [x] Load the CSV extracts into Snowflake `RAW`
- [x] Build 26 dbt staging views
- [x] Build the ten analytical marts
- [ ] Commit the Power BI report, or document its data model and measures if it stays out of the repo
- [ ] Move the Snowflake credentials out of `scripts/upload_to_snowflake.py` into environment variables
- [ ] Fix the loader's CSV path so it reads `data/csv/` directly
- [ ] Add dbt tests and a `profiles.yml.example`
- [ ] Remove or clearly archive the older `dbt_files/` draft

See the [open issues](https://github.com/Elouahabi-Naoufal/school-data-warehouse-powerbi/issues) for a full list of proposed features (and known issues).

<p align="right">(<a href="#readme-top">back to top</a>)</p>

## Contributing

Contributions are what make the open source community such an amazing place to learn, inspire, and create. Any contributions you make are **greatly appreciated**.

If you have a suggestion that would make this better, please fork the repo and create a pull request. You can also simply open an issue with the tag "enhancement".
Don't forget to give the project a star! Thanks again!

1. Fork the Project
2. Create your Feature Branch (`git checkout -b feature/AmazingFeature`)
3. Commit your Changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the Branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

Keep model changes inside `dw_ecole/`, and update `docs/Tables_Schema_of_Marts.md` when a mart's
columns change.

### Top contributors:

<a href="https://github.com/Elouahabi-Naoufal/school-data-warehouse-powerbi/graphs/contributors">
  <img src="https://contrib.rocks/image?repo=Elouahabi-Naoufal/school-data-warehouse-powerbi" alt="contrib.rocks image" />
</a>

<p align="right">(<a href="#readme-top">back to top</a>)</p>

## License

This project is not licensed — all rights reserved.

<p align="right">(<a href="#readme-top">back to top</a>)</p>

## Contact

Naoufal Elouahabi - [LinkedIn](https://www.linkedin.com/in/elouahabi-naoufal/) - elouahabi-naoufal1@gmail.com

Project Link: [https://github.com/Elouahabi-Naoufal/school-data-warehouse-powerbi](https://github.com/Elouahabi-Naoufal/school-data-warehouse-powerbi)

<p align="right">(<a href="#readme-top">back to top</a>)</p>

## Acknowledgments

* [dbt Documentation](https://docs.getdbt.com/)
* [Snowflake Documentation](https://docs.snowflake.com/)
* [Power BI Documentation](https://learn.microsoft.com/power-bi/)
* [pandas Documentation](https://pandas.pydata.org/docs/)
* [Img Shields](https://shields.io)
* [Choose an Open Source License](https://choosealicense.com)
* [Best-README-Template](https://github.com/othneildrew/Best-README-Template)

<p align="right">(<a href="#readme-top">back to top</a>)</p>

[contributors-shield]: https://img.shields.io/github/contributors/Elouahabi-Naoufal/school-data-warehouse-powerbi.svg?style=for-the-badge
[contributors-url]: https://github.com/Elouahabi-Naoufal/school-data-warehouse-powerbi/graphs/contributors
[forks-shield]: https://img.shields.io/github/forks/Elouahabi-Naoufal/school-data-warehouse-powerbi.svg?style=for-the-badge
[forks-url]: https://github.com/Elouahabi-Naoufal/school-data-warehouse-powerbi/network/members
[stars-shield]: https://img.shields.io/github/stars/Elouahabi-Naoufal/school-data-warehouse-powerbi.svg?style=for-the-badge
[stars-url]: https://github.com/Elouahabi-Naoufal/school-data-warehouse-powerbi/stargazers
[issues-shield]: https://img.shields.io/github/issues/Elouahabi-Naoufal/school-data-warehouse-powerbi.svg?style=for-the-badge
[issues-url]: https://github.com/Elouahabi-Naoufal/school-data-warehouse-powerbi/issues
[linkedin-shield]: https://img.shields.io/badge/-LinkedIn-black.svg?style=for-the-badge&logo=linkedin&colorB=555
[linkedin-url]: https://www.linkedin.com/in/elouahabi-naoufal/
[Python-badge]: https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white
[Python-url]: https://www.python.org/
[Snowflake-badge]: https://img.shields.io/badge/Snowflake-29B5E8?style=for-the-badge&logo=snowflake&logoColor=white
[Snowflake-url]: https://www.snowflake.com/
[dbt-badge]: https://img.shields.io/badge/dbt-FF694B?style=for-the-badge&logo=dbt&logoColor=white
[dbt-url]: https://www.getdbt.com/
[PowerBI-badge]: https://img.shields.io/badge/Power%20BI-F2C811?style=for-the-badge&logo=powerbi&logoColor=black
[PowerBI-url]: https://powerbi.microsoft.com/
[pandas-badge]: https://img.shields.io/badge/pandas-150458?style=for-the-badge&logo=pandas&logoColor=white
[pandas-url]: https://pandas.pydata.org/
[Jupyter-badge]: https://img.shields.io/badge/Jupyter-F37626?style=for-the-badge&logo=jupyter&logoColor=white
[Jupyter-url]: https://jupyter.org/
[SQL-badge]: https://img.shields.io/badge/SQL-4479A1?style=for-the-badge&logo=mysql&logoColor=white
[SQL-url]: https://en.wikipedia.org/wiki/SQL
