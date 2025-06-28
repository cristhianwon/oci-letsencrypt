FROM fnproject/ruby:2.7-dev as build-stage
WORKDIR /function
ADD Gemfile* /function/
RUN microdnf update -y && \
    microdnf install --enablerepo=ol8_codeready_builder -y \
    libyaml-devel gcc make redhat-rpm-config && \
    microdnf clean all
RUN bundle install --jobs 4 --retry 3
FROM fnproject/ruby:2.7
WORKDIR /function
COPY --from=build-stage /usr/lib/ruby/gems/ /usr/lib/ruby/gems/
COPY . /function/
RUN chmod -R o+r /function
ENTRYPOINT ["ruby", "func.rb"]
