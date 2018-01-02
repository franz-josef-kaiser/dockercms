FROM nginx:stable-alpine

# Install rc-update, rc-service
RUN apk add --no-cache openrc

# Replace default main conf
# Symlink `sites-available` to `sites-enabled`
RUN rm /etc/nginx/conf.d/default.conf \
	&& ln -s /etc/nginx/sites-available /etc/nginx/sites-enabled