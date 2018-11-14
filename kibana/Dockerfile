ARG VERSION

# https://github.com/elastic/kibana-docker
FROM docker.elastic.co/kibana/kibana-oss:${VERSION}

# Add your kibana plugins setup here
# Example: RUN kibana-plugin install <name|url>
RUN kibana-plugin install https://github.com/datasweet/kibana-datasweet-formula/releases/download/v2.0.0/datasweet_formula-2.0.0_kibana-6.4.2.zip

