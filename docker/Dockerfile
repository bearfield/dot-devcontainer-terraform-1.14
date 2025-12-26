FROM ghcr.io/bearfield/debian-fish:bookworm
#
# set argument
ARG TERRAFORM_VERSION=1.14.*
#
# Add HashiCorp Linux repository and install terraform.
USER root
SHELL ["/bin/bash", "-c"]
RUN apt-get update \
    && curl -fsSL https://apt.releases.hashicorp.com/gpg | gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg \
    && echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | tee /etc/apt/sources.list.d/hashicorp.list \
    && apt-get update \
    && apt-get install -y \
        terraform=$TERRAFORM_VERSION \
    # install tflint
    && curl -s https://raw.githubusercontent.com/terraform-linters/tflint/master/install_linux.sh | bash \
    ## install trivy
    && apt-get install -y \
        wget \
        gnupg \
    && wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | gpg --dearmor | sudo tee /usr/share/keyrings/trivy.gpg > /dev/null \
    && echo "deb [signed-by=/usr/share/keyrings/trivy.gpg] https://aquasecurity.github.io/trivy-repo/deb generic main" | sudo tee -a /etc/apt/sources.list.d/trivy.list \
    && sudo apt-get update \
    && sudo apt-get install -y \
        trivy \
    # crean up
    && apt-get autoremove -y \
    && apt-get clean -y \
	&& rm -rf /var/lib/apt/lists/*
#
# set env
ENV DEBIAN_FRONTEND=dialog
LABEL org.opencontainers.image.source="https://github.com/bearfield/dot-devcontainer-terraform-1.14"
