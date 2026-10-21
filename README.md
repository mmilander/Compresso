# Compresso
## Project summary
### One-sentence description of the project
Compresso is a secure web-based video compression service that lets users upload videos through a cloud-hosted website while the actual compression runs on an isolated computer behind a home network.
### Additional information about the project
Large videos recorded on phones and computers can be difficult to upload or share because of file-size limits. Compressing them locally also takes processing power and time. Compresso makes this easier by letting users upload a video, select a codec, and choose a target file size from a browser.
The system separates the public website from the machine doing the compression. An Oracle Cloud Infrastructure (OCI) virtual private server (VPS) hosts the website, handles logins, and manages uploaded files and queued jobs. A local rendering laptop connects outward to the VPS through an encrypted WireGuard tunnel and pulls jobs for processing. This means the home network does not need to expose inbound ports to the public internet, including when it is behind NAT or carrier-grade NAT.
For security, FFmpeg processes uploaded videos in a fresh Docker container inside a KVM virtual machine. The application uses Argon2id password hashing, parameterized database queries, checks that users can access only their own jobs, and validates compression settings before running FFmpeg. Temporary files are removed after processing or downloading, with an additional cleanup process for older files.
## Installation
### Prerequisites
Compresso currently uses a two-part setup rather than a single locally installed application:
- Cloud server: A Linux-based Oracle Cloud Infrastructure VPS (or equivalent), with Python, FastAPI, Uvicorn, and the project's application dependencies.
- Rendering machine: A Linux host capable of running KVM virtual machines, with libvirt configured to manage the rendering VM.
- Inside the rendering VM: WireGuard, Docker, and the project's job-processing components. FFmpeg runs in the per-job Docker container.
- Network and security: A configured WireGuard connection between the rendering VM and cloud VPS, a shared secret for authorized requests, and HTTPS configured on the public website.
- Development tools: Git and access to the project's source repository.
- Client: A web browser to access the Compresso website.
Exact software versions and operating-system requirements have not yet been documented.
### Add-ons
- FastAPI and Uvicorn: Runs the Python web application on the cloud VPS.
- Oracle Cloud Infrastructure: Hosts the public website, login handling, and job queue.
- WireGuard: Creates the encrypted connection between the cloud VPS and rendering VM.
- KVM and libvirt:	Provides a separate virtual machine for the rendering environment.
- Docker: Runs each compression job in a fresh container.
- FFmpeg: Compresses uploaded videos with the selected settings.
- Argon2id: Hashes account passwords.
### Installation Steps
TODO: Describe the installation process (making sure you mention `bundle install`).
Instructions need to be such that a user can just copy/paste the commands to get
things set up and running.
## Functionality
A typical Compresso job works as follows:
1. Log in. Open the cloud-hosted Compresso website over HTTPS and sign in. The current development implementation has two hardcoded test accounts.
2. Upload a video. Use the upload area to choose a video file from your computer.
3. Choose compression settings. Select a supported codec (the current interface shows H.264 CPU) and enter the desired target size in MB.
4. Submit the job. Click Compress. The cloud VPS stores the upload and puts the job in a queue.
5. Process the video. The rendering VM retrieves the queued job through WireGuard. FFmpeg compresses the file in a new Docker container inside the KVM guest.
6. Retrieve the result. The compressed video travels back through the tunnel and becomes available for download from the website.
7. Clean up temporary files. The system removes temporary data after jobs and downloads, and runs periodic cleanup for files older than ten minutes.
The goal is to keep the upload process simple while moving CPU-intensive work to a separate computer without exposing the home machine directly to the internet.
## Known Problems
- Self-signed HTTPS certificate: The public website currently uses a self-signed certificate, so browsers may display a certificate warning. The connection is encrypted, but the certificate is not automatically trusted for authenticating the server. The plan is to use a certificate issued by a trusted authority.
- Test-only accounts: Login currently supports two hardcoded test users rather than a completed account-management system.
- Container network access: Compression containers still have network access. Disabling network access for the FFmpeg container is a planned security improvement.
- Missing container resource limits: CPU and memory limits for the compression containers have not yet been configured. A malicious or problematic input could consume excessive resources.
- Tunnel and reconnect behavior (needs confirmation): The milestone report says the system may need to be restarted if the rendering VM disconnects or changes networks, but its description of VPS forwarding conflicts with the described pull-based job flow. Confirm the actual behavior in code before treating this as a verified bug.
- Render-side firewall hardening: Additional restrictions are planned to ensure the cloud side of the tunnel cannot initiate unwanted connections to the rendering environment.
- Performance results not yet documented: The milestone report focuses on the architecture and security improvements but does not include measured compression times or before-and-after file sizes. This does not necessarily mean those tests have not been performed; their results still need to be documented.
## Contributing
TODO: Leave the steps below if you want others to contribute to your project.
1. Fork it!
2. Create your feature branch: `git checkout -b my-new-feature`
3. Commit your changes: `git commit -am 'Add some feature'`
4. Push to the branch: `git push origin my-new-feature`
5. Submit a pull request :D
## Additional Documentation
TODO: Provide links to additional documentation that may exist in the repo, e.g.,
* Sprint reports
* User links
## License
If you haven't already, add a file called `LICENSE.txt` with the text of the
appropriate license.
We recommend using the MIT license: <https://choosealicense.com/licenses/mit/>
