#
# Copyright 2025 Staysail Systems, Inc. <info@staysail.tech>
# Copyright 2018 Capitar IT Group BV <info@capitar.com>
#
# This software is supplied under the terms of the MIT License, a
# copy of which should be located in the distribution where this
# file was obtained (LICENSE.txt).  A copy of the license may also be
# found online at https://opensource.org/licenses/MIT.
#

# Surveyor/Respondent protocol
nng_directory(survey0)

nng_sources_if(NNG_PROTO_SURVEYOR0 survey.c xsurvey.c)
nng_defines_if(NNG_PROTO_SURVEYOR0 NNG_HAVE_SURVEYOR0)

nng_sources_if(NNG_PROTO_RESPONDENT0 respond.c xrespond.c)
nng_defines_if(NNG_PROTO_RESPONDENT0 NNG_HAVE_RESPONDENT0)

nng_test(respond_test)
nng_test(survey_test)
nng_test(xrespond_test)
nng_test(xsurvey_test)
