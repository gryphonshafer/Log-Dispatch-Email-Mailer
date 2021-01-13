# NAME

Log::Dispatch::Email::Mailer - Log::Dispatch::Email subclass that sends mail using Email::Mailer

# VERSION

version 1.10

[![test](https://github.com/gryphonshafer/Log-Dispatch-Email-Mailer/workflows/test/badge.svg)](https://github.com/gryphonshafer/Log-Dispatch-Email-Mailer/actions?query=workflow%3Atest)
[![codecov](https://codecov.io/gh/gryphonshafer/Log-Dispatch-Email-Mailer/graph/badge.svg)](https://codecov.io/gh/gryphonshafer/Log-Dispatch-Email-Mailer)

# SYNOPSIS

    use Log::Dispatch;

    # simple text email alert via Log::Dispatch
    my $log = Log::Dispatch->new(
        outputs => [
            [
                'Email::Mailer',
                min_level => 'alert',
                to        => [ qw( foo@example.com bar@example.org ) ],
                subject   => 'Alert Log Message',
            ],
        ],
    );
    $log->alert('This is to alert you something happened.');

    # simple text email alert via direct instantiation
    my $email = Log::Dispatch::Email::Mailer->new(
        min_level => 'alert',
        to        => [ qw( foo@example.com bar@example.org ) ],
        subject   => 'Alert Log Message',
    );
    $email->log(
        message => 'This is to alert you something happened.',
        level   => 'alert',
    );

    # simple text email using an Email::Mailer object with explicit transport
    $log = Log::Dispatch->new(
        outputs => [
            [
                'Email::Mailer',
                min_level => 'alert',
                to        => [ qw( foo@example.com bar@example.org ) ],
                subject   => 'Alert Log Message',
                mailer    => Email::Mailer->new(
                    transport => Email::Sender::Transport::SMTP->new({
                        host => 'smtp.example.com',
                        port => 25,
                    }),
                ),
            ],
        ],
    );
    $log->alert('This is to alert you something happened.');

    # HTML email alert with attached log file using Template Toolkit
    use Template;
    my $tt = Template->new;
    $log   = Log::Dispatch->new(
        outputs => [
            [
                'Email::Mailer',
                min_level => 'alert',
                to        => [ qw( foo@example.com bar@example.org ) ],
                subject   => 'Alert Log Message',
                html      => \q{
                    <pre>[% message %]</pre>
                    <p>[% messages.join("<br>") %]</p>
                },
                attachments => [
                    {
                        ctype   => 'text/plain',
                        content => 'This is plain text attachment content.',
                        name    => 'log_file.txt',
                    },
                ],
                process => sub {
                    my ( $template, $data ) = @_;
                    my $content;
                    $tt->process( \$template, $data, \$content );
                    return $content;
                },
            ],
        ],
    );
    $log->alert('This is to alert you something happened.');

# DESCRIPTION

This is a subclass of [Log::Dispatch::Email](https://metacpan.org/pod/Log%3A%3ADispatch%3A%3AEmail) that implements the `send_email()`
method using the [Email::Mailer](https://metacpan.org/pod/Email%3A%3AMailer) module. Much like the [Email::Mailer](https://metacpan.org/pod/Email%3A%3AMailer) module,
you can send email in a great variety of ways including text-only, HTML with
text auto-generated, including attachments, and even using your favorite
templating system.

## Simple Text Email

The simplest way to use this module is to setup an "outputs" record with
[Log::Dispatch](https://metacpan.org/pod/Log%3A%3ADispatch) much like you would any other email subclass.

    my $log = Log::Dispatch->new(
        outputs => [
            [
                'Email::Mailer',
                min_level => 'alert',
                to        => [ qw( foo@example.com bar@example.org ) ],
                subject   => 'Alert Log Message',
            ],
        ],
    );
    $log->alert('This is to alert you something happened.');

By default, log messages are buffered and sent either when `$log` is destroyed
or when you call `$log->flush`.

    $log->alert('This message will appear in an email.');
    $log->alert('This message will appear in the same email, but not yet...');
    $log->flush; # now both alerts will get sent in one email

Note that unlike many other [Log::Dispatch::Email](https://metacpan.org/pod/Log%3A%3ADispatch%3A%3AEmail) subclasses, multiple
buffered messages won't be concatenated together without spaces. Instead, the
messages will appear in a text-only email as independent lines.

As an alternative to buffering, you can explicitly set buffering off to have
each log line send a single email.

    my $log = Log::Dispatch->new(
        outputs => [
            [
                'Email::Mailer',
                min_level => 'alert',
                to        => [ qw( foo@example.com bar@example.org ) ],
                subject   => 'Alert Log Message',
                buffer    => 0,
            ],
        ],
    );
    $log->alert('This will be in one email.');
    $log->alert('This will be in a second email.');

## Simple Text Email with Explicit Transport

By default, this module will create its own [Email::Mailer](https://metacpan.org/pod/Email%3A%3AMailer) object through
which to send email. You can provide a "mailer" value of an explicit
[Email::Mailer](https://metacpan.org/pod/Email%3A%3AMailer) object you create and control, thus allowing you to set things
like an explicit transport mechanism.

    my $log = Log::Dispatch->new(
        outputs => [
            [
                'Email::Mailer',
                min_level => 'alert',
                to        => [ qw( foo@example.com bar@example.org ) ],
                subject   => 'Alert Log Message',
                mailer    => Email::Mailer->new(
                    transport => Email::Sender::Transport::SMTP->new({
                        host => 'smtp.example.com',
                        port => 25,
                    }),
                ),
            ],
        ],
    );
    $log->alert('This is to alert you something happened.');

## HTML Email with Attached File Using Template Toolkit

If you want to have some real fun with sending email log messages (and let's be
real here, who doesn't), try using this module to send templated HTML email
with attachments. Any key/value you can pass to [Email::Mailer](https://metacpan.org/pod/Email%3A%3AMailer), you can pass
as part of the "outputs" element.

The following example uses an HTML template (which per [Email::Mailer](https://metacpan.org/pod/Email%3A%3AMailer) needs
to be a scalar reference) and a very simple Template Toolkit process subref.

    use Template;
    my $tt  = Template->new;
    my $log = Log::Dispatch->new(
        outputs => [
            [
                'Email::Mailer',
                min_level => 'alert',
                to        => [ qw( foo@example.com bar@example.org ) ],
                subject   => 'Alert Log Message',
                html      => \q{
                    <pre>[% message %]</pre>
                    <p>[% messages.join("<br>") %]</p>
                },
                attachments => [
                    {
                        ctype   => 'text/plain',
                        content => 'This is plain text attachment content.',
                        name    => 'log_file.txt',
                    },
                ],
                process => sub {
                    my ( $template, $data ) = @_;
                    my $content;
                    $tt->process( \$template, $data, \$content );
                    return $content;
                },
            ],
        ],
    );
    $log->alert('This is to alert you something happened.');

What's happening behind the scenes is that the "data" value that you'd normally
pass to [Email::Mailer](https://metacpan.org/pod/Email%3A%3AMailer) that would work its way down into the "process" subref
is in this case being generated for you. It gets populated with two sub-keys:
message and messages. The first is a "\\n"-separated string of log messages.
The second is an arrayref of those strings.

# SEE ALSO

[Email::Mailer](https://metacpan.org/pod/Email%3A%3AMailer), [Log::Dispatch::Email](https://metacpan.org/pod/Log%3A%3ADispatch%3A%3AEmail), [Log::Dispatch](https://metacpan.org/pod/Log%3A%3ADispatch).

You can also look for additional information at:

- [GitHub](https://github.com/gryphonshafer/Log-Dispatch-Email-Mailer)
- [MetaCPAN](https://metacpan.org/pod/Log::Dispatch::Email::Mailer)
- [GitHub Actions](https://github.com/gryphonshafer/Log-Dispatch-Email-Mailer/actions)
- [Codecov](https://codecov.io/gh/gryphonshafer/Log-Dispatch-Email-Mailer)
- [CPANTS](http://cpants.cpanauthors.org/dist/Log-Dispatch-Email-Mailer)
- [CPAN Testers](http://www.cpantesters.org/distro/D/Log-Dispatch-Email-Mailer.html)

# AUTHOR

Gryphon Shafer <gryphon@cpan.org>

# COPYRIGHT AND LICENSE

This software is Copyright (c) 2017-2021 by Gryphon Shafer.

This is free software, licensed under:

    The Artistic License 2.0 (GPL Compatible)
