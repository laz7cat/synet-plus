### [Experimental] Docker Installation
As an interim step before modernizing the code (including migrating to Python 3), we dockerize it with a reproducible working environment, including Python 2.7 and a
Z3 solver pinned to commit
[`a63d1b184800954aef888fb76d531237f574f957`](https://github.com/Z3Prover/z3/commit/a63d1b184800954aef888fb76d531237f574f957). Example usage (mounts current host directory to container):
```
docker build -t synet-plus .
docker run --rm -it -v "$(pwd)":/workspace/synet-plus -w /workspace/synet-plus synet-plus bash
mkdir -p out/bgp_peers
python synet/examples/bgp_peers.py out/bgp_peers
```


### Install from Source - The following have been tested with Python 2.7

1. Install Z3 from master, see https://github.com/Z3Prover/z3 (**message from Sep 2025**: The current version of z3 will not work with this version of NetComplete. You might want to try using a commit from when the codebase was tested, such as the one that follows from Dec 2018)

```
git clone https://github.com/Z3Prover/z3.git /tmp/z3
cd /tmp/z3
git checkout a63d1b184800954aef888fb76d531237f574f957
python scripts/mk_make.py --python
cd build
make -j"$(nproc)"
make install
ldconfig
cd /
rm -rf /tmp/z3
```

2. Install network graph library dependency first

```
pip install -e git+git@github.com:nsg-ethz/tekton.git#egg=Tekton
# Or from a local clone
pip install -e .
```

(**message from Sep 2025**: To resolve an incompatibility between Python 2.7 and the ipaddress library used in Tekton -`cisco.py` file-, I also had to perform the following change as a quick workaround:)
```
sed -i "s/ip_network(str(network))/ip_network(unicode(network))/g" \
    /usr/local/lib/python2.7/site-packages/tekton/cisco.py
```

3. Install python dependencies

```
pip install -r requirements.txt
```

### An Example use of NetComplete

The example at `synet/examples/bgp_peers.py` shows how to use NetComplete to synthesize Provider/Customer peering policies.

Running
```
python synet/examples/bgp_peers.py outdir
```

### Running NSDI Experiements

Running BGP experiements
```
# BGP
 ./eval_scripts/run-ebgp-experiments.sh
# OSPF
 ./eval_scripts/run-ospf-experiments.sh
```

### Running OSPF
```python synet/drivers/ospf_driver.py --help```

Example

```bash
python synet/drivers/ospf_driver.py  -p [NUMBER OF PATHs generated per iteration] -r [NUMBER OF REQS] -f [TOPO FILE NAME.graphml]
```

```bash
python synet/drivers/ospf_driver.py -r 20 -p 100 -f topos/topozoo/AttMpls.graphml
```


### Running Tests

Commonly used ```nosetests``` options:

- Timing each test case: ```--with-timer```
- Running specific tests with tags ```-a tag=value```
- Running specific test case ```nosetests FILE_PATH:TESTCLASS``` or ```nosetests FILE_PATH:TESTCLASS.TESTCASE```

Running fast tests:
```nosetests --with-timer -a speed=fast test/```

Running slow tests:
```nosetests --with-timer -a speed=slow test/```

Running all tests (fast and slow)
```nosetests --with-timer test```
