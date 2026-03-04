# ─────────────────────────────────────────────
#  PAYMENT APP — Dockerfile
#  Image: nginx serving static HTML
#  Build:   docker build -t payment-app .
#  Run:     docker run -p 8083:80 payment-app
# ─────────────────────────────────────────────

FROM nginx:alpine

# Remove default nginx page
RUN rm -rf /usr/share/nginx/html/*

# Copy our payment page
COPY index.html /usr/share/nginx/html/index.html
COPY nginx.conf /etc/nginx/conf.d/default.conf

# Expose port 80
EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
