# vulnextract

Pipeline untuk mengubah dataset pasangan kode CVE (vulnerable vs fixed) menjadi dataset tabular (CSV) pada granularitas **method** atau **statement**, sebagai bahan riset deteksi kerentanan perangkat lunak. Mendukung Go, Python, JavaScript, TypeScript, PHP, Java, Rust, dan C — parsing per bahasa menggunakan [tree-sitter](https://tree-sitter.github.io/tree-sitter/).

## Prasyarat

- Python 3.10+
- pip

## Instalasi

Dari root repo:

```bash
python -m venv .venv
```

Aktifkan virtual environment:

```bash
# Windows (PowerShell)
.venv\Scripts\Activate.ps1

# Windows (cmd)
.venv\Scripts\activate.bat

# Linux / macOS
source .venv/bin/activate
```

Install dependency:

```bash
pip install -r requirements.txt
```

## Menjalankan pipeline

Entry point ada di [src/main.py](src/main.py):

```bash
python src/main.py
```

Perintah di atas otomatis membaca `data/input/cve_fix_pairs.csv` dan menulis hasilnya ke `data/output/output_csv_fix_pairs.csv` dengan granularitas `statement` dan strategi pairing `aligned` (nilai default).

### Opsi CLI

| Opsi | Deskripsi | Default |
|---|---|---|
| `-i`, `--input` | Path file CSV input | `data/input/cve_fix_pairs.csv` |
| `-o`, `--output` | Path file CSV output | `data/output/output_csv_fix_pairs.csv` |
| `-g`, `--granularity` | `statement` atau `method` | `statement` |
| `-s`, `--strategy` | Strategi pairing: `aligned` atau `strict` | `aligned` |
| `-v`, `--verbose` | Aktifkan log debug | nonaktif |

### Contoh

Ekstraksi per statement (default):

```bash
python src/main.py -i data/input/cve_fix_pairs.csv -o data/output/output_csv_fix_pairs.csv
```

Ekstraksi per method, dengan pairing ketat (skip baris kalau jumlah method vulnerable ≠ fixed):

```bash
python src/main.py -g method -s strict
```

Ke file custom, dengan log verbose:

```bash
python src/main.py -i data/input/my_dataset.csv -o data/output/my_result.csv -v
```

## Format data

### Input

CSV dengan kolom minimal berikut (lihat [data/input/cve_fix_pairs.csv](data/input/cve_fix_pairs.csv)):

`cve_id, vulnerability_type, language, file, method, vulnerable_code, fixed_code, commit_hash, repo, commit_msg`

Satu baris merepresentasikan satu method/fungsi (vulnerable dan fixed) dari satu commit perbaikan CVE. Nilai `language` harus salah satu dari: `go`, `python`, `javascript`, `typescript`, `php`, `java`, `rust`, `c` (case-insensitive; beberapa alias seperti `py`, `js`, `ts`, `golang`, `rs`, `cpp` juga didukung — lihat [src/extractors/__init__.py](src/extractors/__init__.py)).

### Output

Kolom sama seperti input, ditambah kolom `granularity` (`method`/`statement`). Untuk granularitas `statement`, satu baris input method bisa menghasilkan banyak baris output — satu per statement yang berhasil dipasangkan antara versi vulnerable dan fixed.

## Menjalankan test

```bash
python -m unittest tests.test_pipeline -v
```

## Struktur proyek

```
src/
  main.py            # CLI entry point
  config.py          # path default input/output
  pipeline.py         # orkestrasi: baca CSV -> extract -> pairing -> tulis CSV
  models.py           # dataclass/enum (CodePair, PairingResult, PipelineStats)
  extractors/          # satu extractor tree-sitter per bahasa
  strategies/pairing.py # strategi pairing vulnerable<->fixed (aligned/strict)
tests/
  test_pipeline.py    # unit test extractor, pairing, dan pipeline
data/
  input/               # dataset CVE mentah
  output/              # hasil ekstraksi
context.md             # panduan kolom fitur (rencana pengembangan lanjutan)
```

## Catatan

- `extract_statements` pada tiap extractor mengambil statement langsung dari body function/method via AST (bukan split baris kosong), sehingga hasilnya stabil walau gaya format (jumlah baris kosong) berbeda antara versi vulnerable dan fixed.
- Kolom-kolom fitur keamanan (sink berbahaya, guard, dsb.) yang dijelaskan di [context.md](context.md) masih berupa rencana/panduan — belum diimplementasikan di pipeline ini.
