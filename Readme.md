# VMCK

## Installation

1. Set up a local nomad cluster:

    * Install nomad:
        ```shell
        mkdir /tmp/cluster; cd /tmp/cluster
        curl -OL https://releases.hashicorp.com/nomad/0.8.7/nomad_0.8.7_linux_amd64.zip
        unzip nomad_0.8.7_linux_amd64.zip
        ```

    * Create a configuration file, `/tmp/cluster/nomad.hcl`, with the following
      content, replacing `eth0` with the default network interface, which you
      can determine by running `route | grep default | awk '{print $8}'`:
        ```hcl
        advertise {
          http = "{{ GetInterfaceIP `eth0` }}"
          serf = "{{ GetInterfaceIP `eth0` }}"
        }
        server {
          job_gc_threshold = "5m"
        }
        client {
          enabled = true
          network_interface = "eth0"
        }
        ```

    * Run nomad:
        ```shell
        cd /tmp/cluster
        ./nomad agent -dev -config=./nomad.hcl &
        ```

2. TODO explain how to set up an http server for the images

3. Set up the VMCK server:
    ```shell
    pipenv install
    echo 'SECRET_KEY=changeme' > .env
    echo 'DEBUG=true' >> .env
    echo 'QEMU_IMAGE_URL=http://example.com/artful.qcow2' >> .env
    pipenv run ./manage.py migrate
    ```

## Usage

* Run the web server:
    ```shell
    pipenv run ./manage.py runserver
    ```

* Create a job:
    ```shell
    pipenv run ./manage.py createjob
    ```

## Testing
With a Nomad cluster running on `localhost:4646`, run `pipenv run pytest`, and
enjoy. To make the tests run faster you can mirror the VM image locally and
override the URL in the local `.env` file - see `testsuite/settings.py` for the
default image.

## Troubleshooting
* QEMU fails to start with error `qemu-system-x86_64: Invalid host forwarding
  rule 'tcp:${attr.unique.network.ip-address}:10674-:22' (Bad host address)`:
  the host address has changed (e.g. because it moved to a different WiFi
  hotspot). Restart Nomad and it should pick up the new address.
