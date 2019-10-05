# NAME

Mojolicious::Plugin::ModeHelpers - Mojolicious Plugin that adds helpers to determine the mode and avoid typos

# STATUS

<div>
    <a href="https://travis-ci.org/srchulo/Mojolicious-Plugin-ModeHelpers"><img src="https://travis-ci.org/srchulo/Mojolicious-Plugin-ModeHelpers.svg?branch=master"></a>
</div>

# SYNOPSIS

    use Mojolicious::Lite;
    plugin 'ModeHelpers';

    if (app->in_prod) { # true if app->mode eq 'production'
      # do prod stuff
    } elsif (app->in_dev) { # true if app->mode ne 'production'
      # do dev stuff
    }

    # rename in_prod and in_dev
    plugin ModeHelpers => { prod_helper_name => 'in_production_mode', dev_helper_name => 'in_development_mode' };
    if (app->in_production_mode) { # true if app->mode eq 'production'
      # do prod stuff
    } elsif (app->in_development_mode) { # true if app->mode ne 'production'
      # do dev stuff
    }

    # provide your own custom modes to generate helpers
    plugin ModeHelpers => { modes => ['alpha', 'beta'] };
    if (app->in_alpha) { # true if app->mode eq 'alpha'
      # do alpha stuff
    } elsif (app->in_beta) { # true if app->mode eq 'beta'
      # do beta stuff
    }

    # weird modes get valid perl subroutine names
    plugin ModeHelpers => { modes => ['my strange mode!'] };
    if (app->in_my_strange_mode) { # true if app->mode eq 'my strange mode!'
      # do strange things
    }

    # provide your own helper name and mode pairs
    plugin ModeHelpers => { modes => [{ in_alpha_mode => 'alpha' }, 'beta'] };
    if (app->in_alpha_mode) { # true if app->mode eq 'alpha'
      # do alpha stuff
    } elsif (app->in_beta) { # true if app->mode eq 'beta'
      # do beta stuff
    }

    # use Mojolicious helper dot notation
    plugin ModeHelpers => {
      prod_helper_name => 'modes.prod',
      dev_helper_name => 'modes.dev',
      modes => [{ 'modes.alpha' => 'alpha' }]
    };

    if (app->modes->prod) { # true if app->mode eq 'production'
      # do prod stuff
    }
    if (app->modes->dev) { # true if app->mode ne 'production'
      # do dev stuff
    }

    if (app->modes->alpha) { # true if app->mode eq 'alpha'
      # do alpha stuff
    }

# DESCRIPTION

[Mojolicious::Plugin::ModeHelpers](https://metacpan.org/pod/Mojolicious::Plugin::ModeHelpers) is a [Mojolicious::Plugin](https://metacpan.org/pod/Mojolicious::Plugin) that adds helpers so that you can know what mode
you are in via a method call instead of comparing to the string returned by ["mode" in Mojolicious](https://metacpan.org/pod/Mojolicious#mode). This can help with typos, and
is often more compact. You may use the built-in ["in\_prod"](#in_prod) and ["in\_dev"](#in_dev) methods, or you can add helpers for your custom ["modes"](#modes).

# METHODS

## in\_prod

Returns true if ["mode" in Mojolicious](https://metacpan.org/pod/Mojolicious#mode) is `production`. Otherwise, returns false.

    if (app->in_prod) { # true if app->mode eq 'production'
      # do prod stuff
    }

["in\_prod"](#in_prod) can be renamed via ["prod\_helper\_name"](#prod_helper_name).

## in\_dev

Returns true if ["mode" in Mojolicious](https://metacpan.org/pod/Mojolicious#mode) does not equal `production`. Otherwise, returns false.

    if (app->in_dev) { # true if app->mode ne 'production'
      # do dev stuff
    }

["in\_dev"](#in_dev) can be renamed via ["dev\_helper\_name"](#dev_helper_name).

## register

    my $config = $plugin->register($app);
    my $config = $plugin->register($app, { modes => ['alpha', 'beta', 'gamma'] });

Register plugin in [Mojolicious](https://metacpan.org/pod/Mojolicious) application and create helpers.

# OPTIONS

## prod\_helper\_name

["prod\_helper\_name"](#prod_helper_name) allows you to change the name of the ["in\_prod"](#in_prod) helper:

    plugin ModeHelpers => { prod_helper_name => 'in_production_mode' };

    if (app->in_production_mode) { # true if app->mode eq 'production'
      # do prod stuff
    }

You can also use the ["helper" in Mojolicious](https://metacpan.org/pod/Mojolicious#helper) dot notation:

    plugin ModeHelpers => { prod_helper_name => 'modes.prod' };

    if (app->modes->prod) {
      # do prod stuff
    }

## dev\_helper\_name

["dev\_helper\_name"](#dev_helper_name) allows you to change the name of the ["in\_dev"](#in_dev) helper:

    plugin ModeHelpers => { dev_helper_name => 'in_development_mode' };

    if (app->in_development_mode) { # true if app->mode ne 'production'
      # do dev stuff
    }

You can also use the ["helper" in Mojolicious](https://metacpan.org/pod/Mojolicious#helper) dot notation:

    plugin ModeHelpers => { dev_helper_name => 'modes.dev' };

    if (app->modes->dev) {
      # do dev stuff
    }

## mode

["modes"](#modes) allows you to pass in custom modes that will have their own helpers that return true
if ["mode" in Mojolicious](https://metacpan.org/pod/Mojolicious#mode) equals their mode. Modes can either be a non-empty scalar, or a hash that has
a key-value pair of helper\_name => mode.

    plugin ModeHelpers => {
      modes => [
        'alpha', # generates helper in_alpha for mode 'alpha'
        { in_beta_mode => 'beta' }, # generates helper in_beta_mode for mode 'beta'
        'my strange mode!', # generates helper in_my_strange_mode for mode 'my strange mode!'
      ],
    };

    if (app->in_alpha) { # true if app->mode eq 'alpha'
      # do alpha stuff
    } elsif (app->in_beta_mode) { # true if app->mode eq 'beta'
      # do beta stuff
    } elsif (app->in_my_strange_mode) { # true if app->mode eq 'my strange mode!'
      # do strange things
    }

### SCALAR

Non-empty strings can be passed into ["modes"](#modes). The helper name is generated with these steps:

- Pass the mode to ["slugify" in Mojo::Util](https://metacpan.org/pod/Mojo::Util#slugify).
- Replace all dashes with underscores.
- Append the resulting value to the string "in\_".

    plugin ModeHelpers => { modes => ['alpha', 'my strange mode!'] };

    if (app->in_alpha) { # true if app->mode eq 'alpha'
      # do alpha stuff
    } elsif (app->in_my_strange_mode) { # true if app->mode eq 'my strange mode!'
      # do strange things
    }

### HASH

A key-value pair can be provided as a hash, where the key is the helper name and the value is the mode.

    plugin ModeHelpers => { modes => [ { in_alpha_mode => 'alpha' } ] };

    if (app->in_alpha_mode) { # true if app->mode eq 'alpha'
      # do alpha stuff
    }

You can also use the ["helper" in Mojolicious](https://metacpan.org/pod/Mojolicious#helper) dot notation:

    plugin ModeHelpers => { modes => [ { 'modes.alpha' => 'alpha' } ] };

    if (app->modes->alpha) { # true if app->mode eq 'alpha'
      # do alpha stuff
    }

# AUTHOR

Adam Hopkins <srchulo@cpan.org>

# COPYRIGHT

Copyright 2019- Adam Hopkins

# LICENSE

This library is free software; you can redistribute it and/or modify
it under the same terms as Perl itself.

# SEE ALSO
