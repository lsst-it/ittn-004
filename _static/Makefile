TEMPFILE := $(shell mktemp -u)

all: resolvers.png

%.png : %.dot
	dot -Tpng $< -o $@

clean:
	rm *.png
