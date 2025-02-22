# multi-dbench
Benchmark Kubernetes persistent disk volumes with [fio](https://fio.readthedocs.io/en/latest/)

# Usage

1. Build the docker image using the Dockerfile supplied and upload to your local repo. This will pick up the latest version of FIO.
2. Edit `dbench-statefulset.yaml`, updating <br>
	(a) the container/repo location, as built above (under containers/image) <br>
	(b) to keep things simple, the example uses the default storage class; either define a storageclass as required or set a default sc to match your env <br>
	(c) finally update the number of replicas required <br>
3. Deploy Dbench using: `kubectl apply -f dbench-statefulset.yaml`
4. Once deployed, each POD will run a daemon of fio, thus creating a set of worker nodes. 
5. Run a FIO client to run jobs on the worker nodes, see notes below
6. Once the tests are finished, clean up using: `kubectl delete -f dbench-statefulset.yaml` and delete the volumes `k get pvc | awk '/dbench/{print $1}' | xargs -I {} kubectl delete pvc {}` 


## Using FIO

### FIO CLIENT

You can either login to one of the PODs (recommended) or create VM to test with. <br>
If running via a POD, skip to step 5

For the VM, follow steps 1-4 below <br>
NOTE: pods must have routable networking if using a VM

1. Download an Ubuntu (Jammy) image
2. Install requirements: `sudo apt install gcc make zlib1g-dev`
3. Clone FIO: `git clone https://github.com/axboe/fio.git`
4. Make & install FIO: 

```
cd fio
./configure
make
sudo make install
```

### RUNNING TESTS

5. Define an FIO file (see example.fio)

6. Obtain the IP addresses of the pods, e.g.: <br>
`kubectl get pods -o wide | awk '/multi-dbench/{print $6}'`

7. Log into one of the PODs or the FIO VM 

8. Use VI to create two new text files, one containing the FIO parameters (xxx.fio) and the other a list of IP addresses (fio_workers)

9. Run FIO with the IP list and parameter file name, e.g.: `fio --client=fio_workers example.fio`


### NOTES

* If the Persistent Volume Claim is stuck on Pending, check the default storageclass (or add the storageclass to the manifest)
* It can take some time for a Persistent Volume to be Bound
* When cloning ensure that the `docker-entrypoint.sh` has EXECUTE permissions

## Contributors

* Lee Liu (LogDNA)
* [Alexis Turpin](https://github.com/alexis-turpin)
* [Dharmesh Bhatt](https://github.com/darkmesh-b)

## License

* MIT
