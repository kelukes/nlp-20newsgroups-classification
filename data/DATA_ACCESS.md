# Data Access

This project uses the original full 20 Newsgroups archive, not the scikit-learn packaged loader.

Dataset file:

```text
20news-19997.tar.gz
```

Download command:

```bash
mkdir -p data/raw

curl -L \
  -o data/raw/20news-19997.tar.gz \
  https://qwone.com/~jason/20Newsgroups/20news-19997.tar.gz

tar -xzf data/raw/20news-19997.tar.gz -C data/raw
```

The project selects seven categories from the extracted archive:

```text
comp.graphics
comp.os.ms-windows.misc
comp.sys.ibm.pc.hardware
comp.sys.mac.hardware
comp.windows.x
sci.crypt
sci.electronics
```

The raw archive is excluded from the repository to avoid storing unnecessary raw data. Processed files, split assignments, configuration files, metrics, and key outputs are saved in `data/processed/` and `outputs/`.
