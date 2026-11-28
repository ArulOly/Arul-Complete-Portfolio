# Native examples and category audit

The portfolio contains 23,000 project READMEs across 23 categories. This review checked the main embedded implementation for each project and identified where native tools would make the category clearer.

## Tested native starting points

| Category | Example | Verification |
|---|---|---|
| Ubuntu | [Package inventory with dpkg-query](Ubuntu/Project_002001/README.md) | Ubuntu 24.04 container |
| Debian | [Package inventory with dpkg-query](Debian/Project_003001/README.md) | Debian 12 container |
| Fedora | [Package inventory with rpm](Fedora/Project_004001/README.md) | Fedora 43 container |
| FreeBSD | [Base system inventory with freebsd-version](FreeBSD/Project_005001/README.md) | FreeBSD 14.2 VM |
| PostgreSQL | [Temporary tables and a SQL reorder query](PostgreSQL/Project_025001/README.md) | PostgreSQL 17 |
| MySQL | [Temporary tables and a SQL reorder query](MySQL/Project_026001/README.md) | MySQL 8.4 |
| SQLite | [In-memory inventory and SQL constraints](SQLite/Project_028001/README.md) | SQLite engine |
| MongoDB | [Inventory aggregation in mongosh](MongoDB/Project_027001/README.md) | MongoDB 8.0 and mongosh |
| Redis | [Inventory hash and Lua script](Redis/Project_045001/README.md) | Redis 7 |
| AWS | [S3 inventory declaration in CloudFormation](AWS/Project_014001/README.md) | Local cfn-lint validation |
| Azure | [Storage inventory declaration in ARM JSON](Azure/Project_015001/README.md) | Storage account resource checked against the ARM schema |
| Google Cloud | [Bucket inventory declaration in Terraform](Google%20Cloud/Project_016001/README.md) | Local HCL parsing and configuration assertions |

The cloud examples describe configurations. Live cloud deployment and provider-side behavior were outside this verification.

## Other categories reviewed

| Categories | Main implementation | Finding |
|---|---|---|
| Python, FastAPI, Flask, Jupyter, Scikit-learn | Python | Python fits the language, framework, or notebook runtime. |
| Java, JavaScript, TypeScript, Node.js | Java, JavaScript, or TypeScript | The examples already use the category's main language and runtime. |
| HTML/CSS | HTML and CSS | The examples already use browser-native markup and styles. |
| Power BI | Power Query M | The examples already use M. Execution in Power BI Desktop remains unverified. |

## Remaining modernization

The twelve categories in the first table each have 1,000 projects. This change replaces the first inventory implementation in each category. The other 11,988 READMEs in those categories still contain their existing Python-based examples.

The remaining work can be grouped by what each platform does well.

- **Operating systems.** Use shell utilities and platform-specific tools for package inventory, logs, services, file integrity, and scheduled jobs.
- **Databases.** Put aggregation, validation, and reporting in SQL, MongoDB aggregation, or Redis commands and Lua. Keep client code for input or presentation where it serves a purpose.
- **Cloud providers.** Use CloudFormation, ARM or Bicep, and Terraform to define resources. For workflow guides, use provider-specific configuration or tooling and document which checks ran locally.
