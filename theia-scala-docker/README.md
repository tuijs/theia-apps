# Theia Scala

### Build

```bash
docker build . -t theiaide/theia-scala:latest
```

### Run

```bash
docker run -it --init -p 3000:3000 -v "$(pwd):/home/project" theiaide/theia-scala:latest
```

