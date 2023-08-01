# multi-dbench
Benchmark Kubernetes persistent disk volumes with [fio](https://fio.readthedocs.io/en/latest/)
Forked from https://github.com/leeliu/dbench

# Usage

1. Build the docker image using the Dockerfile supplied and upload to your local repo. This will pick up the latest version of FIO.
2. Edit `dbench-statefulset.yaml`, inserting the container location and changing `storageClassName` to match your Kubernetes provider's Storage Class `kubectl get storageclasses`. Optionally update the number of replicaas required
3. Deploy Dbench using: `kubectl apply -f dbench-statefulset.yaml`
4. Once deployed, the Dbench Job will run a daemon of fio, thus creating a set of worker nodes. NOTE: pods must have routable networking
5. Run a FIO client to run jobs on the worker nodes, see notes below
6. Once the tests are finished, clean up using: `kubectl delete -f dbench-statefulset.yaml` and delete the volumes `k get pvc | awk '/dbench/{print $1}' | xargs -I {} kubectl delete pvc {}` 

## Notes / Troubleshooting

### FIO CLIENT

You can either login to one of the PODs or create VM to test with

For the VM, follow steps 1-4 below (or skip to step 5 if executing via a POD)

1. Download an Ubuntu image.
2. Install requirements: `sudo apt install gcc make zlib1g-dev`
3. Clone FIO: `git clone https://github.com/axboe/fio.git`
4. Make & install FIO: 

```
cd fio
./configure
make
sudo make install
```

5. Define an FIO file (see example.fio)
6. Obtain the IP addresses of the pods, e.g.: <br>
`seq 0 15 | xargs -I {} bash -c 'kubectl get pod multi-dbench-{} --template '{{.status.podIP}}';echo' > fio_workers`

7. Run FIO, e.g.: `fio --client=fio_workers example.fio`

* If the Persistent Volume Claim is stuck on Pending, it's likely you didn't specify a valid Storage Class. Double check using `kubectl get storageclasses`.
* It can take some time for a Persistent Volume to be Bound

## Contributors

* Lee Liu (LogDNA)
* [Alexis Turpin](https://github.com/alexis-turpin)
* [Dharmesh Bhatt](https://github.com/darkmesh-b)

## License

* MIT
