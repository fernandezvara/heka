.. _config_docker_log_input:

Docker Log Input
================

.. versionadded:: 0.8

Plugin Name: **DockerLogInput**

The DockerLogInput plugin attaches to all containers running on a host and
sends their logs messages into the Heka pipeline. The plugin is based on
`Logspout <https://github.com/progrium/logspout>`_ by Jeff Lindsay.
Messages will be populated as follows:

- Uuid: Type 4 (random) UUID generated by Heka.
- Timestamp: Time when the log line was received by the plugin.
- Type: `DockerLog`.
- Hostname: Hostname of the machine on which Heka is running.
- Payload: The log line received from a Docker container.
- Logger: `stdout` or `stderr`, depending on source.
- Fields["ContainerID"] (string): The container ID
- Fields["ContainerName"] (string): The container name

.. note::

	Logspout expects to be dealing exclusively with textual log file data, and
	always assumes that the file data is newline delimited, i.e. one line in
	the log file equals one logical unit of data. For this reason, the
	DockerLogInput currently does *not* support the use of alternate splitter
	plugins. Any splitter setting specified in a DockerLogInput's
	configuration will be ignored.

Config:

- endpoint (string):
    A Docker endpoint. Defaults to "unix:///var/run/docker.sock".
- decoder (string):
    The name of the decoder used to further transform the message into a
    structured hekad message. No default decoder is specified.

.. versionadded:: 0.9

- cert_path (string, optional):
    Path to directory containing client certificate and keys. This value works
    in the same way as `DOCKER_CERT_PATH <https://docs.docker.com/articles/https/#client-modes>`_.

.. versionadded:: 0.10

- name_from_env_var(string):
		Overwrite the ContainerName with this environment variable on the Container
		if exists.

Example:

.. code-block:: ini

   [nginx_log_decoder]
   type = "SandboxDecoder"
   filename = "lua_decoders/nginx_access.lua"

   [nginx_log_decoder.config]
   type = "nginx.access"
   user_agent_transform = true
   log_format = '$remote_addr - $remote_user [$time_local] "$request" $status $body_bytes_sent "$http_referer" "$http_user_agent"'

   [DockerLogInput]
   decoder = "nginx_log_decoder"
