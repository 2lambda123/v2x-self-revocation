# https://stackoverflow.com/a/23324703
ROOT_DIR:=$(shell dirname $(realpath $(firstword $(MAKEFILE_LIST))))

DOCKER_NAME ?= selfrevocation/prl_size
DOCKER_BINDING = -v $(ROOT_DIR):/usr/src/app -t

# Set DOCKER_CMD to the empty string to work locally
DOCKER_CMD ?= docker run --rm $(DOCKER_BINDING) $(DOCKER_NAME)

CACHE_DIR ?= cached
PLOT_DIR ?= plots

MKDIR ?= mkdir -m 777 -p

build-docker:
	docker build . -t $(DOCKER_NAME)

test: init_dirs
	$(DOCKER_CMD) python3 main.py -n 3 -p 0.01 -e 2 --cache-dir=$(CACHE_DIR) --allow-cached

probabilities:
	$(DOCKER_CMD) python3 probabilities.py

single: init_dirs
	$(DOCKER_CMD) python3 main.py -n 800 -p 0.000000116323325 -e 30 --cache-dir=$(CACHE_DIR) --allow-cached

tikz: init_dirs
	@echo "Generating transition graph (Fig. 15)."
	$(DOCKER_CMD) python3 main.py -n 3 -p 0.1 -e 2 -f $(PLOT_DIR)/tikz-graph.tex

p-plot: init_dirs
	$(DOCKER_CMD) ./scripts/p-plot.sh --cache-dir $(CACHE_DIR) --path $(PLOT_DIR)

n-plot: init_dirs
	$(DOCKER_CMD) ./scripts/n-plot.sh --cache-dir $(CACHE_DIR) --path $(PLOT_DIR)

t-plot: init_dirs
	$(DOCKER_CMD) ./scripts/t-plot.sh --cache-dir $(CACHE_DIR) --path $(PLOT_DIR)

tv-distribution: init_dirs
	$(DOCKER_CMD) ./scripts/tv-distribution.sh --cache-dir $(CACHE_DIR) --path $(PLOT_DIR)

.PHONY: check
check:
	@if [ -d "$(ROOT_DIR)/$(CACHE_DIR)" ]; then \
		echo "Cached Directory exists. Continuing under the assumption that you know this might take a while."; \
	else \
		echo "!!! Cache dir does not exist!\n!!! This means that \`make all\` will take a long time. We suggest to start small and run \`make test\` first and check the Readme.\n!!! Continue anyway? [y/n]"; \
	read line; \
	if [ ! $$line = "y" ]; then \
		echo aborting; \
		exit 1; \
	else \
		echo okay, continuing. This might take a while..; \
	fi ; \
	fi

all: init_dirs check probabilities p-plot t-plot n-plot tv-distribution tikz
	@echo Successfully created all graphs!

init_dirs:
	@$(MKDIR) $(CACHE_DIR)
	@$(MKDIR) $(PLOT_DIR)
	@$(MKDIR) $(PLOT_DIR)/distributions

clean:
	rm -rf $(CACHE_DIR) $(PLOT_DIR)