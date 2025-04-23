FROM fluent/fluentd:v1.16-debian

USER root

RUN gem install fluent-plugin-grok-parser --no-document \
    && gem install fluent-plugin-elasticsearch --no-document \
    && gem install fluent-plugin-rewrite-tag-filter --no-document \
    && gem install fluent-plugin-tag-normaliser --no-document \ 
    && gem install fluent-plugin-docker_metadata_filter --no-document 
USER root