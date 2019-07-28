FROM centos:centos7 as builder
ENV build_date 2019-04-25

RUN yum update -y && \
    yum install -y epel-release && \
    yum install -y \
        git \
        patch \
        kernel-headers \
        gcc \
        gcc-c++ \
        cpp \
        ncurses-devel \
        libxml2-devel \
        sqlite-devel \
        openssl-devel \
        newt-devel \
        kernel-devel \
        libuuid-devel \
        net-snmp-devel \
        libedit-devel \
        xinetd \
        tar \
        jansson-devel \
        make \
        bzip2 \
        pjproject-devel \
        libsrtp-devel \
        gsm-devel \
        speex-devel \
        gettext \
      && yum clean all \
      && rm -rf /var/cache/yum/*

# Download asterisk.
WORKDIR /usr/src
RUN git clone -b 16.5 --depth 1 https://github.com/asterisk/asterisk.git

# Configure
RUN mkdir /opt/asterisk
WORKDIR /usr/src/asterisk
RUN ./configure --prefix=/opt/asterisk/ --with-jansson-bundled

RUN make menuselect.makeopts
RUN menuselect/menuselect \
  --disable BUILD_NATIVE \
  --enable cdr_csv \
  --enable res_snmp \
  --enable res_http_websocket \
  --enable res_hep_pjsip \
  --enable res_hep_rtcp \
  --enable res_sorcery_astdb \
  --enable res_sorcery_config \
  --enable res_sorcery_memory \
  --enable res_sorcery_memory_cache \
  --enable res_pjproject \
  --enable res_rtp_asterisk \
  --enable res_ari \
  --enable res_ari_applications \
  --enable res_ari_asterisk \
  --enable res_ari_bridges \
  --enable res_ari_channels \
  --enable res_ari_device_states \
  --enable res_ari_endpoints \
  --enable res_ari_events \
  --enable res_ari_mailboxes \
  --enable res_ari_model \
  --enable res_ari_playbacks \
  --enable res_ari_recordings \
  --enable res_ari_sounds \
  --enable res_pjsip \
  --enable res_pjsip_acl \
  --enable res_pjsip_authenticator_digest \
  --enable res_pjsip_caller_id \
  --enable res_pjsip_config_wizard \
  --enable res_pjsip_dialog_info_body_generator \
  --enable res_pjsip_diversion \
  --enable res_pjsip_dlg_options \
  --enable res_pjsip_dtmf_info \
  --enable res_pjsip_empty_info \
  --enable res_pjsip_endpoint_identifier_anonymous \
  --enable res_pjsip_endpoint_identifier_ip \
  --enable res_pjsip_endpoint_identifier_user \
  --enable res_pjsip_exten_state \
  --enable res_pjsip_header_funcs \
  --enable res_pjsip_logger \
  --enable res_pjsip_messaging \
  --enable res_pjsip_mwi \
  --enable res_pjsip_mwi_body_generator \
  --enable res_pjsip_nat \
  --enable res_pjsip_notify \
  --enable res_pjsip_one_touch_record_info \
  --enable res_pjsip_outbound_authenticator_digest \
  --enable res_pjsip_outbound_publish \
  --enable res_pjsip_outbound_registration \
  --enable res_pjsip_path \
  --enable res_pjsip_pidf_body_generator \
  --enable res_pjsip_publish_asterisk \
  --enable res_pjsip_pubsub \
  --enable res_pjsip_refer \
  --enable res_pjsip_registrar \
  --enable res_pjsip_registrar_expire \
  --enable res_pjsip_rfc3326 \
  --enable res_pjsip_sdp_rtp \
  --enable res_pjsip_send_to_voicemail \
  --enable res_pjsip_session \
  --enable res_pjsip_sips_contact \
  --enable res_pjsip_t38 \
  --enable res_pjsip_transport_management \
  --enable res_pjsip_transport_websocket \
  --enable res_pjsip_xpidf_body_generator \
  --enable res_stasis \
  --enable res_stasis_answer \
  --enable res_stasis_device_state \
  --enable res_stasis_mailbox \
  --enable res_stasis_playback \
  --enable res_stasis_recording \
  --enable res_stasis_snoop \
  --enable res_stasis_test \
  --enable res_statsd \
  --enable res_timing_timerfd \
  menuselect.makeopts

# Continue with a standard make.
RUN make 1> /dev/null && make install 1> /dev/null && make samples 1> /dev/null
RUN rm -rf /usr/src/asterisk

# Second Stage#
FROM centos:centos7
ENV build_date 2019-04-25
ENV PATH="/opt/asterisk/sbin:${PATH}"

RUN yum update -y && \
    yum install -y epel-release && \
    yum install -y \
        libxml2 \
        sqlite \
        openssl-devel \
        newt-devel \
        libuuid-devel \
        libedit \
        net-snmp-devel \
        xinetd \
        tar \
        jansson-devel \
        pjproject-devel \
        libsrtp-devel \
        gsm-devel \
        speex-devel \
        gettext \
      && yum clean all \
      && rm -rf /var/cache/yum/*

WORKDIR /

COPY --from=builder /opt/asterisk/ /opt/asterisk/
# Update max number of open files.
RUN sed -i -e 's/# MAXFILES=/MAXFILES=/' /opt/asterisk/sbin/safe_asterisk

# Copy in default configs
COPY http.conf /opt/asterisk/etc/asterisk/http.conf

RUN localedef -i en_US -f UTF-8 en_US.UTF-8

# And run asterisk in the foreground.
CMD /opt/asterisk/sbin/asterisk -f
