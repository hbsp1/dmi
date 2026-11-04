    '''
    ETL setup and process:
    1. Setup config.yaml
        dir: /dmi/src/database/config.yaml

        example:
            db_config:
            name: dmi_db
            user: postgres
            password: 1234
            host: localhost
            port: 5433 

    3. Setup database:
        3.1 Run docker from terminal:
            docker run --name postgres_dmi -e POSTGRES_PASSWORD=1234 -e POSTGRES_DB=dmi_db -p 5433:5432 -d postgres
        
        3.2 Run def create_db_schema(conn_str)
            dir: /dmi/src/database/init_postgres_docker.py

    4. Run main.py

1. Create py project environment
    python -m venv .venv

2. Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass (let PS run scripts)

3. .\.venv\Scripts\Activate.ps1

4. pip install pandas openpyxl psycopg[binary] pyyaml requests



# Project Title

Simple overview of use/purpose.

## Description

An in-depth paragraph about your project and overview of use.

## Getting Started

### Dependencies

* Describe any prerequisites, libraries, OS version, etc., needed before installing program.
* ex. Windows 10

### Installing

* How/where to download your program
* Any modifications needed to be made to files/folders

### Executing program

* How to run the program
* Step-by-step bullets
```
code blocks for commands
```

## Help

Any advise for common problems or issues.
```
command to run if program contains helper info.
```

## Authors

Contributors names and contact info.

ex. Dominique Pizzie  
ex. [@DomPizzie](https://twitter.com/dompizzie)

## Version History

* 0.2
    * Various bug fixes and optimizations
    * See [commit change]() or See [release history]()
* 0.1
    * Initial Release

## License

This project is licensed under the [NAME HERE] License - see the LICENSE.md file for details

## Acknowledgments

Inspiration, code snippets, etc.
* [awesome-readme](https://github.com/matiassingers/awesome-readme)
* [PurpleBooth](https://gist.github.com/PurpleBooth/109311bb0361f32d87a2)
* [dbader](https://github.com/dbader/readme-template)
* [zenorocha](https://gist.github.com/zenorocha/4526327)
* [fvcproductions](https://gist.github.com/fvcproductions/1bfc2d4aecb01a834b46)